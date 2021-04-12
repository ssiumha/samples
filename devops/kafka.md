# KAFKA

## LISTENER 설정하기 (with docker)

- https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/
- https://www.confluent.io/blog/kafka-listeners-explained/


```
KAFKA_LISTENERS: INDOCKER://0.0.0.0:49092,LOCAL://0.0.0.0:9092
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INDOCKER:PLAINTEXT,LOCAL:PLAINTEXT
KAFKA_ADVERTISED_LISTENERS: INDOCKER://host.docker.internal:49092,LOCAL://localhost:9092
KAFKA_INTER_BROKER_LISTENER_NAME: INDOCKER

KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
KAFKA_DELETE_TOPIC_ENABLE: 'true'
KAFKA_CREATE_TOPICS: topic-test:1:1
```

- KAFKA_ADPERTISED_LISTERS: broker를 가리키는 주소목록, kafka에 최초 연결시 client에게 전달함
- KAFKA_LISTENERS: kafka broker가 요청을 listen 할 주소 (0.0.0.0:9092 등)
- KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER에서 정의된 이름에 protocol을 매핑시킨다
- KAFKA_INTER_BROKER_LISTENER_NAME: 브로커간 통신에 사용할 LISTENER 이름을 가리킴



```
KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE


# bin/kafka-console-producer.sh --broker-list kafka:9093
+----------------- in docker ------------------------------------+
|                                                                |
| producer ----- connect to kafka:9093 ----> kafka               |
|                                                                |
| producer <---- redirect to PLAINTEXT://kafka:9093 ----- kafka  |
|                                                                |
| producer ----- connect to PLAINTEXT://kafka:9093 ----> kafka   |
|                                                                |
| producer ----- produce messages ----> kafka                    |
|                                                                |
+----------------------------------------------------------------+


# bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
                                                        +-- in docker ---+
                                                        |                |
producer ----- connect to localhost:9092 ---------------+---> kafka      |
                                                        |                |
producer <---- redirect to PLAINTEXT://localhost:9092 --+---- kafka      |
                                                        |                |
producer ----- connect to PLAINTEXT://localhost:9092  --+---> kafka      |
                                                        |                |
producer <---- consume message -------------------------+---- kafka      |
                                                        |                |
                                                        +----------------+
```

### `Could not connect to leader for partition development_topic/0: Kafka::LeaderNotAvailable` 이슈

- docker에서 kafka를 띄우고 다른 docker에서 연결시킬 때 발생한 문제
- kafka가 새로 뜰 때마다 broker_id가 변경되지만 zookeeper에서는 예전 broker_id를 기억하며
  leader를 못맞춰서 발생한 문제  `KAFKA_BROKER_ID: 1`로 지정하고 zookeeper를 포함해서 싹다 날리면 해결된다
- zookeeper에 topic key, partition, leader 정보가 함께 있어서 kafka 삭제 만으로는 해결이 되지 않는다

```
# 현재 kafka node는 1030번이지만 node id가 1001, 1028 일때 만들어진 topic 들은 당시 node id가 leader고,
# 새로 뜨면서 날아갔을 기존 leader에 접근하려고 해도 실패하여 LeaderNotAvailable 에러가 발생하게 된다
Topic:development_topic-test     PartitionCount:1        ReplicationFactor:1     Configs:
        Topic: development_topic-test    Partition: 0    Leader: 1028    Replicas: 1028  Isr: 1028
Topic:topic-test        PartitionCount:1        ReplicationFactor:1     Configs:
        Topic: topic-test       Partition: 0    Leader: 1001    Replicas: 1001  Isr: 1001
```


