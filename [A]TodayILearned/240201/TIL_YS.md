# 들어가며

알림 서비스를 SSE 구현 함에 있어 두 가지 문제 상황이 존재했습니다

1. 알림을 캐싱해두고 재연결 시 전송이 되지 않았던 알림들을 보내 줄 수 있어야 한다.
2. FE와 통신 과정에서 GET 요청에 Header가 제대로 담기지 않는다.

## 1번 이슈

### 기존에 구현한 방식 (JVM 활용)

```java
// repository

@Repository
@RequiredArgsConstructor
public class EmitterRepository {
    private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();
    private final Map<String, Object> eventCache = new ConcurrentHashMap<>();
 
    public SseEmitter save(String emitterId, SseEmitter sseEmitter) {
        emitters.put(emitterId,sseEmitter);
        return sseEmitter;
    }
 
    public void saveEventCache(String emitterId, Object event) {
        eventCache.put(emitterId,event);
    }
 
    public Map<String, SseEmitter> findAllEmitters() {
        return new HashMap<>(emitters);
    }
 
    public Map<String, SseEmitter> findAllEmitterStartWithById(String memberId) {
        return emitters.entrySet().stream()
                .filter(entry -> entry.getKey().startsWith(memberId))
                .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
    }
 
    public Map<String, Object> findAllEventCacheStartWithById(String memberId) {
        return eventCache.entrySet().stream()
                .filter(entry -> entry.getKey().startsWith(memberId))
                .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
    }
 
    public void deleteById(String emitterId) {
        emitters.remove(emitterId);
    }
}
```

1. 클라이언트가 구독 요청을 보내면 해당 클라이언트에게 식별자를 부여하고 SSE 객체를 생성하여 JVM에 저장해줍니다.
2. 이벤트가 발생하면 JVM에 캐싱해두고, SSE 연결이 되어있는 상태라면 알림을 보내줍니다.
3. 만약 클라이언트가 구독 요청을 보냈을 때, 마지막으로 수신한 알림 이후 받지 못한 알림이 있다면 JVM에서 조회 후 전송해줍니다.

### 문제점

1. JVM에 무제한으로 캐싱해둘 수 없기 때문에 정리해주는 작업이 필요한데 기준이나 동시성이 모호하다.
2. 서버 재시작 시 데이터가 유실될 수 있다.
3. 현재는 소규모 서비스라 JVM에 SSE 객체를 등록하는데 무리가 없지만 추후 제약이 생겨 알림 서버를 Scale-Out할 경우 각 알림 서버가 제각각 동작할 수 있다.

### 해결점

Redis를 사용하여 이런 문제점들을 극복하고자 하였습니다.

1. Redis TTL을 활용하여 캐싱 기간을 정해두고 저장소를 정리한다.
2. 서버 인메모리를 사용하지 않고 외부 메모리를 사용한다.
3. Redis Pub/Sub 기능을 활용하여 여러 알림 서버들의 메시지 브로커 기능을 하도록 한다.

### Redis 기반으로 구현한 방식

```java
// config
@Configuration
public class RedisConfig {

    @Value("${data.redis.port}")
    private int port;

    @Value("${data.redis.host}")
    private String host;

    @Bean
    public RedisConnectionFactory redisConnectionFactory(){
        return new LettuceConnectionFactory(host, port);
    }

    @Primary
    @Bean
    public RedisTemplate<String, String> redisTemplate(){
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());

        return redisTemplate;
    }
}
```
```java
// service
@Service
@RequiredArgsConstructor
@Slf4j
public class NotificationService {

    private final EmitterRepository emitterRepository;
    private final RedisTemplate<String, String> redisTemplate;
    private final ObjectMapper objectMapper;

    /**
     * Kafka 토픽마다 이벤트 발생 시 SSE 전송 요청
     *
     * @param targetId     SSE 수신할 Emitter key
     * @param kafkaMessage SSE 수신할 내용
     */
    public void kafkaListen(Long targetId, String kafkaMessage) throws JsonProcessingException {

        FollowMessage followMessage = objectMapper.readValue(kafkaMessage, FollowMessage.class);
        long time = System.currentTimeMillis();
        NotificationCache notificationCache = NotificationCache.builder()
                .content(followMessage.content())
                .time(time)
                .build();
        String message = objectMapper.writeValueAsString(notificationCache);
        // Redis 캐싱용 백업 (하루)
        redisTemplate.opsForZSet()
                .add(String.valueOf(targetId),
                        message,
                        time);
        redisTemplate.expire(String.valueOf(targetId), 24, TimeUnit.HOURS);

        Optional<SseEmitter> sseEmitter = emitterRepository.get(targetId);
        if (sseEmitter.isEmpty()) {
            return;
        }

        sendToClient(sseEmitter.get(), targetId, notificationCache);
    }

    /**
     * SSE 구독 처리
     * 동일 요청 사용자가 받은 마지막 알림 이후 들어왔던 알림이 있다면 구독과 동시에 밀린 알림들 전송
     *
     * @param memberId 사용자 member_id 당 Emitter 할당
     * @return SSE Emitter
     */
    public SseEmitter subscribe(Long memberId, String lastMessage) throws JsonProcessingException {

        SseEmitter emitter = emitterRepository.save(memberId, new SseEmitter(3600L * 100));
        emitter.onCompletion(() -> {
            emitterRepository.deleteById(memberId);
        });
        emitter.onTimeout(() -> {
            emitterRepository.deleteById(memberId);
        });
        sendToClient(emitter, memberId, "EventStream Created. [member=" + memberId + "]");
        if (!lastMessage.equals("empty")) {
            Set<String> notificationsByTimeRange = getNotificationsByTimeRange(memberId, Long.valueOf(lastMessage));
            for (String notification : notificationsByTimeRange) {
                NotificationCache notificationCache = objectMapper.readValue(notification, NotificationCache.class);
                sendToClient(emitter, memberId, notificationCache);
            }
        }

        return emitter;
    }

    /**
     * 1분마다 연결 상태 확인
     * TimeOut 과는 다른 상태 확인 메서드
     */
    @Scheduled(fixedRate = 60000)
    public void sendHeartbeat() {
        Map<Long, SseEmitter> emitters = emitterRepository.findAllEmitters();
        emitters.forEach((key, emitter) -> {
            try {
                emitter.send(SseEmitter.event().id(String.valueOf(key)).name("heartbeat").data(""));
            } catch (IOException e) {
                emitterRepository.deleteById(key);
            }
        });
    }

    private void sendToClient(SseEmitter emitter, Long memberId, Object message) {
        try {
            emitter.send(SseEmitter.event()
                    .id(String.valueOf(memberId))
                    .data(message));
        } catch (IOException e) {
            emitterRepository.deleteById(memberId);
        }
    }

    private Set<String> getNotificationsByTimeRange(Long memberId, Long lastMessageTime) {
        long min = lastMessageTime;
        long max = Long.MAX_VALUE;

        return redisTemplate.opsForZSet()
                .rangeByScore(String.valueOf(memberId), min, max);
    }
}
```

코드를 하나씩 살펴보도록 하겠습니다.

1. kafkaListen 메소드

```java
// Redis 캐싱용 백업 (하루)
redisTemplate.opsForZSet()
    .add(String.valueOf(targetId),
            message,
            time);
redisTemplate.expire(String.valueOf(targetId), 24, TimeUnit.HOURS);
```

Redis sorted Set 자료형을 활용하여 구현하였습니다.

이벤트마다 시스템 시간을 부여하여 수신한 마지막 알림 시간을 파악 할 수 있도록 하였습니다.

Redis에 저장할 때는 해당 시간을 sorted Set의 score로 하여 저장하였습니다.

- Redis sorted Set
    - 멤버를 점수에 따라 정렬된 순서로 저장합니다. 이를 통해 멤버 간의 순서를 유지하면서 원하는 범위의 멤버를 조회할 수 있습니다.
    - 멤버를 고유한 값으로 저장합니다.
    -  Sorted Set은 내부적으로 스킵 리스트(skip list)라는 자료구조를 사용하여 멤버를 저장하고 검색합니다. 

2. subscribe 메소드

```java
if (!lastMessage.equals("empty")) {
    Set<String> notificationsByTimeRange = getNotificationsByTimeRange(memberId, Long.valueOf(lastMessage));
    for (String notification : notificationsByTimeRange) {
        NotificationCache notificationCache = objectMapper.readValue(notification, NotificationCache.class);
        sendToClient(emitter, memberId, notificationCache);
    }
}

private Set<String> getNotificationsByTimeRange(Long memberId, Long lastMessageTime) {
    long min = lastMessageTime;
    long max = Long.MAX_VALUE;

    return redisTemplate.opsForZSet()
            .rangeByScore(String.valueOf(memberId), min, max);
}
```

처음 SSE 요청 시에는 lastMessage를 empty로 요청하고 이후 요청에는 수신한 이벤트의 알림 시간을 지속적으로 갱신하며 요청합니다.

getNotificationsByTimeRange 매소드는 Emitter 식별자를 Key로 하고 마지막 이벤트의 알림 시간 이후에 발생한 미수신 알림들을 반환해줍니다.

## 2번 이슈

### EventSource

FE 구현 시에 SSE에 대해 무지한 상태였는데 SSE 구독 요청을 보낼 때 axios로 요청을 보내며 이벤트 수신이 안되는 문제점을 겪었습니다.

알고보니 EventSource를 활용하여 SSE 구독 요청을 보내는 것이었습니다.

```javascript
const eventSource = new EventSource(
    `http://70.12.247.83:8081/notification-service/notification/subscribe/`,
    {
        headers: {
            member_id: {member_id},
            last_message: {last_message}
        }
    }
);
```

하지만 위와 같은 작성은 불가능하였고, EventSource에는 Header를 담을 수 없는 문제점이 있었습니다.

구독 요청 시 필요한 정보들을 반영하기 위해 Header를 담을 수 있는 EventSourcePolyfill을 사용해야 합니다.