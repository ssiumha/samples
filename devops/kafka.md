# KAFKA

- https://sookocheff.com/post/kafka/kafka-in-a-nutshell/
- https://www.popit.kr/kafka-운영자가-말하는-topic-replication/
- https://www.popit.kr/author/peter5236

## /kafka/bin

kafka server의 /kafka/bin 에 kafka 콘솔 스크립트가 있다.

JMX_PORT가 설정되어 있는 상태로 script 실행시 `Port already in use: 5555`가 뜨며 실패한다.
이런 경우 먼저 `unset JMX_PORT`로 무력화시켜줘야한다.

사용 예시는 kube base

### topic

```
kubectl exec -nstorage sts/kafka -it -- bash -c "\
  unset JMX_PORT; \
  /opt/bitnami/kafka/bin/kafka-topics.sh --bootstrap-server kafka:9092 --describe --topic '.*'
";


# env: broker * 5
Topic: notification   PartitionCount: 3       ReplicationFactor: 3    Configs: flush.ms=1000,segment.bytes=1073741824,flush.messages=10000,max.message.bytes=1000012,retention.bytes=1073741824
        Topic: notification   Partition: 0    Leader: 3       Replicas: 3,0,1 Isr: 3,0,1
        Topic: notification   Partition: 1    Leader: 4       Replicas: 4,1,2 Isr: 4,2,1
        Topic: notification   Partition: 2    Leader: 0       Replicas: 0,2,3 Isr: 0,2,3
Topic: __consumer_offsets       PartitionCount: 50      ReplicationFactor: 3    Configs: compression.type=producer,cleanup.policy=compact,flush.ms=1000,segment.bytes=104857600,flush.messages=10000,max.message.bytes=1000012,retention.bytes=1073741824
        Topic: __consumer_offsets       Partition: 0    Leader: 2       Replicas: 2,0,1 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 1    Leader: 0       Replicas: 0,1,2 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 2    Leader: 1       Replicas: 1,2,0 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 3    Leader: 2       Replicas: 2,1,0 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 4    Leader: 0       Replicas: 0,2,1 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 5    Leader: 1       Replicas: 1,0,2 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 6    Leader: 2       Replicas: 2,0,1 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 7    Leader: 0       Replicas: 0,1,2 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 8    Leader: 1       Replicas: 1,2,0 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 9    Leader: 2       Replicas: 2,1,0 Isr: 2,0,1
        Topic: __consumer_offsets       Partition: 10   Leader: 0       Replicas: 0,2,1 Isr: 2,0,1
        ...
```

- kafka-topics.sh의 --topic에는 정규식을 쓸 수 있다

- Partitoin: topic을 읽고 쓰기할 수 있는 queue의 단위
  - partition 3이라면 3개의 broker에 나눠서 데이터를 저장관리하게 된다
  - replica는 partition마다 지정된다
- ReplicationFactor: 처음 topic을 만들 때 지정하는 값. 이 갯수만큼의 broker에 복제본을 저장한다
- Leader / Fallower: == primary / secondary
  - read/write는 leader를 통해서만 발생
- Isr: In Sync Replica. 현재 복제중인 그룹
  - 만약 leader가 죽게 된다면 Isr의 broker 중에 하나가 leader를 잇게 된다
  - 만약 follwer가 leader에 비해 뒤쳐지게 동기화되면 Isr을 박탈당한다
  - Isr이랑 Replicas랑 실질적으로 같은 정보 같은데 따로 있는 이유가 있나???

#### reassign

```
bin/kafka-reassign-partitions.sh --zookeeper zookeeper:2181 --reassignment-json-file rf.json --execute

{
  "version":1,
  "partitions":[
    {"topic":"notification","partition":0,"replicas":[1,2]}
  ]
}
```

- partition 0의 replica를 [1,2] 2개로 수정
- 기존 leader는 맨앞에 오도록해야 리더가 유지된다


### group consumer

```
GROUP               TOPIC        PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                 HOST            CLIENT-ID
notification_gruop1 notification 0          3904            3904            0               xxxx:1-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /10.0.0.001     xxxx:1
notification_gruop1 notification 1          3987            3987            0               xxxx:1-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /10.0.0.002     xxxx:1
notification_gruop1 notification 2          3904            3905            1               xxxx:1-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /10.0.0.003     xxxx:1
```


### config

```
bin/kafka-configs.sh --bootstrap-server kafka:9092 --entity-type brokers --all --describe
```

### misc

```
# partitoin의 크기가 0인 topic 정보 찾기
kubectl exec -nstorage sts/kafka -it -- bash -c "\
  unset JMX_PORT; \
  /opt/bitnami/kafka/bin/kafka-log-dirs.sh --bootstrap-server kafka:9092 --describe
";

## 위 결과 json에 jq 적용
jq -r -c '.brokers[].logDirs[].partitions[] | [ .partition, .size, .offsetLag ] | select(.[1] == 0)'
```


## LISTENER 설정하기 (with docker)

- https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/
- https://www.confluent.io/blog/kafka-listeners-explained/


```
KAFKA_BROKER_ID: 1
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

