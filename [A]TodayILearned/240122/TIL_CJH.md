#

config/server.properties

파일내용 중 일부
# Listener name, hostname and port the broker will advertise to clients.
# If not set, it uses the value for "listeners".
#advertised.listeners=PLAINTEXT://your.host.name:9092
주석풀고 your.host.name:9092에 카프카 서버 IP를 입력

`bin\windows\zookeeper-server-start.bat config\zookeeper.properties`
`bin\windows\kafka-server-start.bat config\server.properties`

> TOPIC 생성

sh ./bin/kafka-topics.sh --create --topic TOPICNAME --bootstrap-server localhost:9092

> 현재 TOPIC 조회

sh ./bin/kafka-topics.sh --list --bootstrap-server localhost:9092

> TOPIC 상세조회

`sh ./bin/kafka-topics.sh --describe --topic TOPICNAME --bootstrap-server localhost:9092`

Producer

`sh ./bin/kafka-console-producer.sh --topic TOPICNAME --bootstrap-server localhost:9092`

Consumer

`$ sh ./bin/kafka-console-consumer.sh --topic TOPICNAME --from-beginning --bootstrap-server localhost:9092`


### Reference

https://itstudy402.tistory.com/59
