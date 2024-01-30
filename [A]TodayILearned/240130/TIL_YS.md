# 들어가며

Member Service와 Notification Service가 다른 서버로 분리 되어있고, 

Notification Service는 모든 알림 서비스를 처리합니다.

## 목표

1. 회원이 SSE 구독 요청
2. A가 B를 팔로우 할 때 B에게 팔로우 알림 전송되도록 구현
3. Kafka에 Member 토픽 생성 
4. Member Service에서 팔로우 이벤트 발생 시 Member 토픽에 이벤트 전송
5. Notification Service에서 Member 토픽 구독
6. B에게 알림 메시지 전송

## 개발 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- Spring-Kafka

## 1. Member Service에서 팔로우 이벤트 발생 시 Kafka에 이벤트 전송

: Member Service가 Producer 역할을 합니다.

```java
@EnableKafka
@Configuration
public class KafkaProducerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String servers;

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```
```java
@Component
@RequiredArgsConstructor
public class KafkaProducer {

    @Value(value = "${message.topic.name}")
    private String topicName;

    private final KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message) {
        kafkaTemplate.send(topicName, message);
    }
}
```
```java
@Transactional
    @Override
    public void memberFollow(Long memberId, Long socialId) throws JsonProcessingException {

        ,,,

        kafkaProducer.sendMessage(
                objectMapper.writeValueAsString(
                        FollowMessage.builder()
                                .targetId(to.getSocialId())
                                .content(from.getNickname() + " 님이 회원님을 팔로우했습니다.")
                                .build()
                )
        );
    }
```

Json 형식으로 Kafka에 전송하고자 했지만 Deserializer 과정에서 지속적인 에러가 발생하여

Object Mapper를 활용하여 String으로 변환 후 Parsing 하도록 하였습니다.

추후 Json 형식을 활용하는 방안을 모색하고자 합니다.

## 2. Notification Service에서 Kafka 이벤트 확인 후 수신자에게 메시지 전송

```java
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String servers;

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, servers);
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "member");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        return new DefaultKafkaConsumerFactory<>(config);
    }

    @Bean(name = "kafkaListener")
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```
```java
@Component
@RequiredArgsConstructor
public class KafkaConsumer {

    private final NotificationService notificationService;
    private final ObjectMapper objectMapper;

    @KafkaListener(topics = "${message.topic.name}", groupId = "member", containerFactory = "kafkaListener")
    public void consumeMessage(String message) throws IOException {
        FollowMessage followMessage = objectMapper.readValue(message, FollowMessage.class);
        notificationService.notify(followMessage.targetId(), followMessage.content());
    }
}
```