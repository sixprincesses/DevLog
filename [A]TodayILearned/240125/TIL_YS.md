# 1. SSE

![Untitled (1)](https://github.com/sixprincesses/DevLog/assets/116432941/5bd4c1b5-a17f-4969-bb39-dd53b0db4fc0)

: 웹 브라우저에서 서버쪽으로 특정 이벤트를 구독하면, 서버에서 해당 이벤트 발생 시 브라우저에 이벤트를 보내주는 방식입니다

장점
- 한 번 연결 요청을 보내면, 연결이 종료될 때까지 재연결 과정 없이 서버에서 웹 브라우저로 계속 데이터를 보낼 수 있습니다

단점
- 단방향 통신
- 최대 동시 접속 횟수 제한

## 1. 고려사항

- 클라이언트 연결량이 많아질 때 어떻게 할 것인가
- 네트워크 문제, 클라이언트의 비정상 종료 등의 에러 처리
- 알림 이벤트 중요도 별 순서 구분 할 것인가

## 2. 로직 구현

### 1. 실습 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok

### 2. 코드

```java
// controller

@RestController
@RequiredArgsConstructor
public class NotificationController {

    private final NotificationService notificationService;

    // 메시지 알림
    @GetMapping(value = "/api/notification/subscribe", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter subscribe(
            @RequestHeader("social_id") Long social_id
    ) throws IOException {

        return notificationService.subscribe(social_id);
    }
}
```

```java
// service

@Service
@RequiredArgsConstructor
public class NotificationService {

    private final EmitterRepository emitterRepository;

    public SseEmitter subscribe(Long socialId) {
        SseEmitter emitter = createEmitter(socialId);

        sendToClient(socialId, "EventStream Created. [userId=" + socialId + "]");
        return emitter;
    }

    public void notify(Long userId, Object event) {
        sendToClient(userId, event);
    }

    private void sendToClient(Long socialId, Object data) {
        SseEmitter emitter = emitterRepository.get(socialId);
        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event().id(String.valueOf(socialId)).name("sse").data(data));
            } catch (IOException exception) {
                emitterRepository.deleteById(socialId);
                emitter.completeWithError(exception);
            }
        }
    }

    private SseEmitter createEmitter(Long socialId) {

        SseEmitter emitter = new SseEmitter();
        emitterRepository.save(socialId, emitter);

        // Emitter가 완료될 때(모든 데이터가 성공적으로 전송된 상태) Emitter를 삭제한다.
        emitter.onCompletion(() -> emitterRepository.deleteById(socialId));
        // Emitter가 타임아웃 되었을 때(지정된 시간동안 어떠한 이벤트도 전송되지 않았을 때) Emitter를 삭제한다.
        emitter.onTimeout(() -> emitterRepository.deleteById(socialId));

        return emitter;
    }

}
```

```java
// repository

@Repository
@RequiredArgsConstructor
public class EmitterRepository {

    private final Map<Long, SseEmitter> emitterMap = new ConcurrentHashMap<>();

    /**
     * 주어진 아이디와 이미터를 저장
     *
     * @param id      - 사용자 아이디.
     * @param emitter - 이벤트 Emitter.
     */
    public void save(Long id, SseEmitter emitter) {
        emitterMap.put(id, emitter);
    }

    /**
     * 주어진 아이디의 Emitter를 제거
     *
     * @param id - 사용자 아이디.
     */
    public void deleteById(Long id) {
        emitterMap.remove(id);
    }

    /**
     * 주어진 아이디의 Emitter를 가져옴.
     *
     * @param id - 사용자 아이디.
     * @return SseEmitter - 이벤트 Emitter.
     */
    public SseEmitter get(Long id) {
        return emitterMap.get(id);
    }
}
```

# 2. Kafka

## 1. Topic

Topic에 Producer가 data를 넣고 Consumer가 data를 받아가는 방식입니다

- 하나의 Topic은 여러 개의 Partition을 가질 수 있습니다
- Partition 내부에는 Queue와 같이 data가 쌓이게 됩니다
- Consumer는 Partition에서 가장 오래된 data 순서대로 가져갑니다
  - Consumer가 data를 가져가도 Partition에서 삭제되지는 않습니다
    
    -> Consumer Group이 다른 Consumer가 `auto.offset.reset = earliest` 로 설정하고 접근 할 경우 동일 data에 대해 접근이 가능합니다
- Partition이 여러 개인 경우 data는 key를 기반으로 Partition을 선택해 들어갑니다
  - key가 null이고, 기본 Partition을 사용하는 경우 Round Robin으로 할당됩니다
  - key가 있고, 기본 Partition을 사용하는 경우 key의 해시 값을 구하고, 특정 Partition에 할당합니다
- Partion의 record는 최대 record 보존 시간(`log.retention.ms`), 최대 record 보존 크기(`log.retention.byte`)에 따라 삭제됩니다

## 2. Broker

Kafka가 설치되어 있는 서버 단위를 말합니다. 보통 3개 이상의 Broker로 구성하여 사용하는 것을 권장합니다

## 3. Replication

- Broker가 3개라고 가정하면, 원본 Partition을 Broker 한개에 할당하면 해당 Partition이 Leader가 됩니다
- 나머지 Broker에 Replication(Follower Partition)을 각각 한개씩 생성할 수 있습니다
- Leader, Follower Partition을 통틀어 ISR(In Sync Replica)라고 볼 수 있습니다
- Leader를 보유한 Broker에 장애가 생겨도 다른 Broker의 Follower가 Leader 역할을 계승할 수 있습니다
- Leader Partition이 Producer의 data를 전달 받는 주체입니다
  - ack
    - 0: Leader Partition에 data를 전송하고 응답값을 받지 않습니다
    - 1: Leader Partition에 data를 전송하고 정상적으로 받았는지 응답값을 받습니다.
    - all: 1 옵션에 추가로 Follower Partition에 복제도 잘 이루어졌는지 응답값을 받습니다.

## 4. Partitioner

Producer가 data를 보내면 Partitioner를 통해서 Broker로 전송됩니다

Partitioner는 어떤 Partition에 넣을지 결정합니다
