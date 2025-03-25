# Kafka & Zookeeper 플랫폼 상세 문서

## 1. Apache Kafka 개요

Apache Kafka는 분산 이벤트 스트리밍 플랫폼으로, 고성능, 고가용성, 확장성을 제공하며 실시간 데이터 파이프라인 구축과 스트리밍 애플리케이션 개발에 사용됩니다. LinkedIn에서 처음 개발되어 현재는 Apache Software Foundation의 오픈 소스 프로젝트로 관리되고 있습니다.

### 1.1 핵심 특징

- **높은 처리량**: 초당 수백만 개의 메시지 처리 가능
- **낮은 지연 시간**: 밀리초 단위의 지연 시간으로 데이터 전달
- **내구성**: 디스크에 데이터 저장으로 안정성 보장
- **확장성**: 클러스터 노드 추가로 수평적 확장 가능
- **내결함성**: 브로커 장애 시에도 데이터 손실 없이 작동
- **분산 아키텍처**: 다수 서버에 걸쳐 데이터 분산 저장 및 처리
- **스트림 처리**: Kafka Streams API를 통한 실시간 데이터 처리

### 1.2 주요 활용 사례

- **메시지 큐잉**: 시스템 간 비동기 메시지 전달
- **로그 집계**: 여러 시스템의 로그 수집 및 중앙화
- **스트림 처리**: 실시간 데이터 변환 및 분석
- **이벤트 소싱**: 상태 변경을 이벤트로 저장하는 패턴
- **활동 추적**: 사용자 행동 및 시스템 활동 모니터링
- **메트릭 수집**: 시스템 모니터링 데이터 수집
- **마이크로서비스 통신**: 서비스 간 비동기 통신 구현

## 2. Zookeeper 개요

Apache ZooKeeper는 분산 시스템을 위한 코디네이션 서비스로, 설정 관리, 네이밍, 동기화, 그룹 서비스 등을 제공합니다. Kafka는 클러스터 상태 관리와 브로커 조정을 위해 ZooKeeper를 사용합니다.

### 2.1 핵심 특징

- **간단한 분산 코디네이션**: 높은 신뢰성을 가진 분산 코디네이션 서비스
- **순차적 일관성**: 클라이언트 요청을 순서대로 처리
- **원자성**: 요청이 성공하거나 실패하거나 중간 상태 없음
- **단일 시스템 이미지**: 서버가 다수여도 클라이언트는 단일 서비스처럼 접근
- **신뢰성**: 다수의 서버가 실패해도 서비스 유지
- **적시성**: 시스템 상태 갱신 시 클라이언트에 빠르게 통지

### 2.2 Kafka에서의 Zookeeper 역할

- **브로커 등록 관리**: Kafka 브로커 등록 및 상태 추적
- **토픽 설정 저장**: 토픽 구성 정보 저장
- **컨트롤러 선출**: 클러스터 컨트롤러 브로커 선정
- **ACL 관리**: 접근 제어 목록 저장
- **파티션 리더 추적**: 각 파티션 리더 정보 관리
- **Kafka 버전 3.0 이후**: Zookeeper 의존성 제거 계획(KIP-500) 진행 중

## 3. Kafka 아키텍처 및 동작 원리

### 3.1 핵심 개념

#### 3.1.1 브로커(Broker)

Kafka 서버로, 메시지를 저장하고 전달하는 역할을 합니다. 클러스터는 여러 브로커로 구성되며, 각 브로커는 고유한 ID를 가집니다.

#### 3.1.2 토픽(Topic)

메시지를 카테고리별로 구분하는 논리적 단위입니다. 각 토픽은 여러 파티션으로 구성될 수 있습니다.

#### 3.1.3 파티션(Partition)

토픽을 여러 부분으로 나눈 것으로, 병렬 처리와 확장성을 제공합니다. 각 파티션은 순서가 보장된 메시지 시퀀스입니다.

#### 3.1.4 복제(Replication)

데이터 손실 방지를 위해 파티션을 여러 브로커에 복제합니다. 복제 팩터는 각 파티션이 몇 개의 복제본을 가질지 지정합니다.

#### 3.1.5 프로듀서(Producer)

메시지를 생성하여 토픽에 게시하는 클라이언트 애플리케이션입니다.

#### 3.1.6 컨슈머(Consumer)

토픽에서 메시지를 구독하고 처리하는 클라이언트 애플리케이션입니다.

#### 3.1.7 컨슈머 그룹(Consumer Group)

하나의 토픽을 공동으로 소비하는 컨슈머 집합입니다. 각 파티션은 그룹 내의 하나의 컨슈머만 소비할 수 있습니다.

#### 3.1.8 오프셋(Offset)

파티션 내 각 메시지의 위치를 나타내는 순차적 ID입니다. 컨슈머는 오프셋을 통해 소비 위치를 추적합니다.

### 3.2 데이터 흐름

1. **프로듀서**가 메시지를 특정 토픽에 게시합니다.
2. 메시지는 해당 토픽의 특정 **파티션**에 기록됩니다.
3. 파티션은 지정된 복제 팩터에 따라 여러 **브로커**에 복제됩니다.
4. 각 파티션에는 리더와 팔로워가 있으며, 모든 읽기/쓰기 요청은 **리더**를 통해 이루어집니다.
5. **컨슈머**는 특정 파티션에서 오프셋을 기준으로 메시지를 가져옵니다.
6. 컨슈머가 그룹의 일부인 경우, 그룹 내 컨슈머 간에 파티션이 균등하게 분배됩니다.

![Kafka Architecture](https://example.com/kafka-architecture.png)

### 3.3 메시지 전달 보장

Kafka는 세 가지 수준의 메시지 전달 보장을 제공합니다:

1. **At most once**: 메시지가 최대 한 번 전달되며, 손실될 수 있습니다.
2. **At least once**: 메시지가 최소 한 번 전달되며, 중복될 수 있습니다.
3. **Exactly once**: 메시지가 정확히 한 번 전달됩니다 (Kafka Streams API나 트랜잭션 API 사용 시).

### 3.4 데이터 보존 정책

Kafka는 다양한 데이터 보존 정책을 지원합니다:

- **시간 기반**: 설정된 시간(예: 7일) 이후 메시지 삭제
- **크기 기반**: 특정 크기에 도달하면 오래된 메시지부터 삭제
- **압축(Compacted)**: 각 키의 최신 값만 유지하고 이전 값은 삭제

## 4. Kafka와 Zookeeper 도커 구성 분석

현재 프로젝트에서 사용 중인 Kafka와 Zookeeper 도커 구성을 살펴보겠습니다:

### 4.1 Zookeeper Dockerfile 분석

```dockerfile
FROM confluentinc/cp-zookeeper:7.4.0

ENV ZOOKEEPER_CLIENT_PORT=2181
ENV ZOOKEEPER_TICK_TIME=2000

EXPOSE 2181

CMD ["/etc/confluent/docker/run"]
```

구성 요소:

- **베이스 이미지**: Confluent Platform의 Zookeeper 7.4.0 이미지 사용
- **환경 변수**:
  - `ZOOKEEPER_CLIENT_PORT`: Zookeeper 클라이언트 포트 설정 (2181)
  - `ZOOKEEPER_TICK_TIME`: 기본 시간 단위 설정 (밀리초 단위, 2000ms = 2초)
- **포트 노출**: 클라이언트 포트 2181 노출
- **실행 명령**: Confluent 제공 스크립트로 Zookeeper 실행

### 4.2 Kafka Dockerfile 분석

```dockerfile
FROM confluentinc/cp-kafka:7.4.0

ENV KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
ENV KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT
ENV KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092
ENV KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092
ENV KAFKA_INTER_BROKER_LISTENER_NAME=PLAINTEXT
ENV KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1

EXPOSE 9092

CMD ["/etc/confluent/docker/run"]
```

구성 요소:

- **베이스 이미지**: Confluent Platform의 Kafka 7.4.0 이미지 사용
- **환경 변수**:
  - `KAFKA_ZOOKEEPER_CONNECT`: Zookeeper 연결 정보
  - `KAFKA_LISTENER_SECURITY_PROTOCOL_MAP`: 리스너별 보안 프로토콜 매핑
  - `KAFKA_ADVERTISED_LISTENERS`: 클라이언트에게 알려지는 리스너 주소
  - `KAFKA_LISTENERS`: Kafka 브로커가 바인딩할 주소
  - `KAFKA_INTER_BROKER_LISTENER_NAME`: 브로커 간 통신에 사용할 리스너
  - `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR`: 오프셋 토픽의 복제 팩터
- **포트 노출**: Kafka 기본 포트 9092 노출
- **실행 명령**: Confluent 제공 스크립트로 Kafka 실행

### 4.3 docker-compose.yml 설정

```yaml
services:
  zookeeper:
    build:
      context: ../..
      dockerfile: docker/kafka/Dockerfile-zookeeper
    container_name: zookeeper_prod
    environment:
      TZ: Asia/Seoul
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    build:
      context: ../..
      dockerfile: docker/kafka/Dockerfile-kafka
    container_name: kafka_prod
    depends_on:
      - zookeeper
    environment:
      TZ: Asia/Seoul
      KAFKA_ZOOKEEPER_CONNECT: zookeeper_prod:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    ports:
      - "9092:9092"
```

주요 설정:

- **Zookeeper와 Kafka 컨테이너 간 의존성 설정**
- **시간대 설정**: 한국 시간대(Asia/Seoul) 사용
- **네트워크 포트 매핑**: 호스트의 포트를 컨테이너의 포트에 매핑
- **Kafka가 Zookeeper 컨테이너를 찾을 수 있도록 연결 정보 제공**
- **단일 노드 설정**: 개발 및 테스트 환경에 적합한 단일 브로커 구성

## 5. Kafka 및 Zookeeper 구성 최적화

### 5.1 Zookeeper 최적화

#### 5.1.1 메모리 설정

```yaml
environment:
  ZOOKEEPER_HEAP_OPTS: "-Xmx512M -Xms512M"
```

JVM 힙 메모리를 적절히 설정하여 성능과 안정성 향상:

- 작은 환경: 512MB ~ 1GB
- 중간 규모: 1GB ~ 2GB
- 대규모: 2GB ~ 4GB (너무 크게 설정하면 GC 지연 가능성)

#### 5.1.2 고급 Zookeeper 설정

```yaml
environment:
  ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: "24" # 24시간마다 자동 정리
  ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: "3" # 스냅샷 3개 유지
  ZOOKEEPER_MAX_CLIENT_CNXNS: "60" # 최대 클라이언트 연결 수
  ZOOKEEPER_SYNC_LIMIT: "5" # 팔로워가 리더와 동기화되는 최대 시간(틱)
  ZOOKEEPER_INIT_LIMIT: "10" # 팔로워가 리더에 연결되는 최대 시간(틱)
```

#### 5.1.3 Zookeeper 앙상블 구성 (프로덕션)

프로덕션 환경에서는 고가용성을 위해 최소 3개의 Zookeeper 인스턴스 권장:

```yaml
services:
  zookeeper1:
    # 기본 설정 동일
    environment:
      ZOOKEEPER_SERVER_ID: 1
      ZOOKEEPER_SERVERS: zookeeper1:2888:3888;zookeeper2:2888:3888;zookeeper3:2888:3888

  zookeeper2:
    # 기본 설정 동일
    environment:
      ZOOKEEPER_SERVER_ID: 2
      ZOOKEEPER_SERVERS: zookeeper1:2888:3888;zookeeper2:2888:3888;zookeeper3:2888:3888

  zookeeper3:
    # 기본 설정 동일
    environment:
      ZOOKEEPER_SERVER_ID: 3
      ZOOKEEPER_SERVERS: zookeeper1:2888:3888;zookeeper2:2888:3888;zookeeper3:2888:3888
```

### 5.2 Kafka 최적화

#### 5.2.1 메모리 설정

```yaml
environment:
  KAFKA_HEAP_OPTS: "-Xmx1G -Xms1G"
```

JVM 힙 메모리 설정:

- 개발 환경: 1GB ~ 2GB
- 프로덕션 환경: 6GB ~ 12GB (트래픽에 따라 조정)

#### 5.2.2 디스크 I/O 최적화

```yaml
environment:
  KAFKA_LOG_FLUSH_INTERVAL_MESSAGES: "10000"
  KAFKA_LOG_FLUSH_INTERVAL_MS: "1000"
  KAFKA_LOG_RETENTION_HOURS: "168" # 7일
  KAFKA_LOG_SEGMENT_BYTES: "1073741824" # 1GB
  KAFKA_LOG_RETENTION_CHECK_INTERVAL_MS: "300000" # 5분
```

#### 5.2.3 네트워크 설정

```yaml
environment:
  KAFKA_NUM_NETWORK_THREADS: "3" # 네트워크 요청 처리 스레드 수
  KAFKA_NUM_IO_THREADS: "8" # 디스크 I/O 스레드 수
  KAFKA_SOCKET_SEND_BUFFER_BYTES: "102400" # 소켓 전송 버퍼
  KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: "102400" # 소켓 수신 버퍼
```

#### 5.2.4 복제 및 파티션 설정

프로덕션 환경을 위한 설정:

```yaml
environment:
  KAFKA_DEFAULT_REPLICATION_FACTOR: "3" # 기본 복제 팩터
  KAFKA_NUM_PARTITIONS: "3" # 기본 파티션 수
  KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: "3" # 오프셋 토픽 복제 팩터
  KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: "3" # 트랜잭션 로그 복제 팩터
  KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: "2" # 트랜잭션 로그 최소 ISR
```

#### 5.2.5 볼륨 마운트 (데이터 영속성)

```yaml
volumes:
  - kafka_data:/var/lib/kafka/data
```

데이터 영속성을 위한 볼륨 마운트 설정.

### 5.3 다중 브로커 설정 (프로덕션)

프로덕션 환경에서는 고가용성과 확장성을 위해 다중 브로커 구성 권장:

```yaml
services:
  kafka1:
    # 기본 설정 동일
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092

  kafka2:
    # 기본 설정 동일
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka2:9092

  kafka3:
    # 기본 설정 동일
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka3:9092
```

## 6. Kafka 활용 사례 및 응용

### 6.1 이벤트 기반 아키텍처 구현

Kafka를 통한 이벤트 기반 마이크로서비스 아키텍처 구현 예시:

#### 6.1.1 이벤트 프로듀서 (Spring Boot)

```java
@Service
public class OrderService {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;

    @Autowired
    private OrderRepository orderRepository;

    @Transactional
    public Order createOrder(OrderRequest request) {
        // 주문 생성 로직
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setProducts(request.getProducts());
        order.setTotalAmount(calculateTotal(request.getProducts()));
        order.setStatus(OrderStatus.PENDING);

        // 주문 저장
        Order savedOrder = orderRepository.save(order);

        // 주문 생성 이벤트 발행
        OrderEvent event = new OrderEvent(
            savedOrder.getId(),
            OrderEventType.CREATED,
            savedOrder.getUserId(),
            savedOrder.getProducts(),
            savedOrder.getTotalAmount(),
            savedOrder.getStatus(),
            LocalDateTime.now()
        );

        kafkaTemplate.send("order-events", savedOrder.getId().toString(), event);

        return savedOrder;
    }

    // 기타 메소드...
}
```

#### 6.1.2 이벤트 컨슈머 (Spring Boot)

```java
@Service
public class InventoryService {

    @Autowired
    private InventoryRepository inventoryRepository;

    @Autowired
    private KafkaTemplate<String, InventoryEvent> kafkaTemplate;

    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void processOrderEvent(OrderEvent event) {
        if (event.getType() == OrderEventType.CREATED) {
            // 재고 확인 및 할당 로직
            boolean allProductsAvailable = checkAndAllocateInventory(event.getProducts());

            // 재고 상태에 따라 이벤트 발행
            InventoryEvent inventoryEvent;
            if (allProductsAvailable) {
                inventoryEvent = new InventoryEvent(
                    event.getOrderId(),
                    InventoryEventType.ALLOCATED,
                    event.getProducts(),
                    LocalDateTime.now()
                );
            } else {
                inventoryEvent = new InventoryEvent(
                    event.getOrderId(),
                    InventoryEventType.INSUFFICIENT,
                    event.getProducts(),
                    LocalDateTime.now()
                );
            }

            kafkaTemplate.send("inventory-events", event.getOrderId().toString(), inventoryEvent);
        }
    }

    private boolean checkAndAllocateInventory(List<OrderProduct> products) {
        // 재고 확인 및 할당 로직 구현
        for (OrderProduct product : products) {
            Inventory inventory = inventoryRepository.findByProductId(product.getProductId());
            if (inventory == null || inventory.getAvailableQuantity() < product.getQuantity()) {
                return false;
            }
        }

        // 재고 할당
        for (OrderProduct product : products) {
            Inventory inventory = inventoryRepository.findByProductId(product.getProductId());
            inventory.setAllocatedQuantity(
                inventory.getAllocatedQuantity() + product.getQuantity()
            );
            inventory.setAvailableQuantity(
                inventory.getAvailableQuantity() - product.getQuantity()
            );
            inventoryRepository.save(inventory);
        }

        return true;
    }
}
```

#### 6.1.3 이벤트 결합 컨슈머 (Order Status Updater)

```java
@Service
public class OrderStatusService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;

    @KafkaListener(topics = "inventory-events", groupId = "order-status-service")
    public void processInventoryEvent(InventoryEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow(() -> new OrderNotFoundException(event.getOrderId()));

        if (event.getType() == InventoryEventType.ALLOCATED) {
            order.setStatus(OrderStatus.CONFIRMED);

            // 주문 상태 업데이트 이벤트 발행
            OrderEvent orderEvent = new OrderEvent(
                order.getId(),
                OrderEventType.CONFIRMED,
                order.getUserId(),
                order.getProducts(),
                order.getTotalAmount(),
                OrderStatus.CONFIRMED,
                LocalDateTime.now()
            );

            kafkaTemplate.send("order-events", order.getId().toString(), orderEvent);
        } else if (event.getType() == InventoryEventType.INSUFFICIENT) {
            order.setStatus(OrderStatus.REJECTED);

            // 주문 상태 업데이트 이벤트 발행
            OrderEvent orderEvent = new OrderEvent(
                order.getId(),
                OrderEventType.REJECTED,
                order.getUserId(),
                order.getProducts(),
                order.getTotalAmount(),
                OrderStatus.REJECTED,
                LocalDateTime.now()
            );

            kafkaTemplate.send("order-events", order.getId().toString(), orderEvent);
        }

        orderRepository.save(order);
    }
}
```

### 6.2 로깅 및 모니터링 파이프라인

#### 6.2.1 로그 수집기 (Logback 설정)

```xml
<appender name="KAFKA" class="com.github.danielwegener.logback.kafka.KafkaAppender">
    <encoder class="com.github.danielwegener.logback.kafka.encoding.JsonEncoder">
        <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
            <jsonFormatter class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
                <prettyPrint>false</prettyPrint>
            </jsonFormatter>
            <timestampFormat>yyyy-MM-dd'T'HH:mm:ss.SSSX</timestampFormat>
            <appendLineSeparator>true</appendLineSeparator>
            <includeThreadName>true</includeThreadName>
            <includeLoggerName>true</includeLoggerName>
        </layout>
    </encoder>

    <topic>application-logs</topic>
    <keyingStrategy class="com.github.danielwegener.logback.kafka.keying.HostNameKeyingStrategy" />
    <deliveryStrategy class="com.github.danielwegener.logback.kafka.delivery.AsynchronousDeliveryStrategy" />

    <producerConfig>bootstrap.servers=kafka:9092</producerConfig>
    <producerConfig>acks=1</producerConfig>
    <producerConfig>linger.ms=500</producerConfig>
    <producerConfig>max.block.ms=5000</producerConfig>
</appender>

<root level="INFO">
    <appender-ref ref="KAFKA" />
</root>
```

#### 6.2.2 로그 처리 서비스 (Kafka Streams)

```java
@SpringBootApplication
public class LogProcessorApplication {

    public static void main(String[] args) {
        SpringApplication.run(LogProcessorApplication.class, args);
    }

    @Bean
    public KStream<String, JsonNode> processLogs(StreamsBuilder builder) {
        KStream<String, JsonNode> logs = builder.stream(
            "application-logs",
            Consumed.with(Serdes.String(), JsonNodeSerde.serdes())
        );

        // 에러 로그 필터링
        KStream<String, JsonNode> errorLogs = logs.filter((key, value) ->
            value.has("level") &&
            value.get("level").asText().equals("ERROR")
        );

        // 경고 로그 필터링
        KStream<String, JsonNode> warningLogs = logs.filter((key, value) ->
            value.has("level") &&
            value.get("level").asText().equals("WARN")
        );

        // 서비스별 로그 분리
        KStream<String, JsonNode>[] serviceLogStreams = logs.branch(
            (key, value) -> value.has("loggerName") &&
                           value.get("loggerName").asText().startsWith("com.example.userservice"),
            (key, value) -> value.has("loggerName") &&
                           value.get("loggerName").asText().startsWith("com.example.orderservice"),
            (key, value) -> value.has("loggerName") &&
                           value.get("loggerName").asText().startsWith("com.example.inventoryservice")
        );

        // 각 서비스별 토픽으로 전송
        serviceLogStreams[0].to("user-service-logs");
        serviceLogStreams[1].to("order-service-logs");
        serviceLogStreams[2].to("inventory-service-logs");

        // 오류 알림을 위한 토픽 전송
        errorLogs.to("alert-logs");

        // 모든 로그를 Elasticsearch 저장용 토픽으로 전송
        logs.to("logs-for-elasticsearch");

        return logs;
    }
}
```

#### 6.2.3 ElasticSearch 싱크 (Kafka Connect)

```json
{
  "name": "elasticsearch-sink",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "logs-for-elasticsearch",
    "connection.url": "http://elasticsearch:9200",
    "type.name": "_doc",
    "key.ignore": "true",
    "schema.ignore": "true",
    "behavior.on.null.values": "ignore",
    "behavior.on.malformed.documents": "warn",
    "write.method": "insert",
    "transforms": "AddTimestamp",
    "transforms.AddTimestamp.type": "org.apache.kafka.connect.transforms.InsertField$Value",
    "transforms.AddTimestamp.timestamp.field": "kafka_ingestion_time"
  }
}
```

### 6.3 실시간 분석 파이프라인

#### 6.3.1 사용자 활동 트래킹 (프론트엔드)

```javascript
// 활동 추적 클라이언트
class ActivityTracker {
  constructor(userId, sessionId) {
    this.userId = userId;
    this.sessionId = sessionId;
    this.baseData = {
      userId,
      sessionId,
      userAgent: navigator.userAgent,
      screenSize: `${window.innerWidth}x${window.innerHeight}`,
    };
  }

  trackPageView(page) {
    this.sendEvent("PAGE_VIEW", {
      page,
      referrer: document.referrer,
      timestamp: new Date().toISOString(),
    });
  }

  trackClick(elementId, elementType) {
    this.sendEvent("CLICK", {
      elementId,
      elementType,
      page: window.location.pathname,
      timestamp: new Date().toISOString(),
    });
  }

  trackSearch(query, results) {
    this.sendEvent("SEARCH", {
      query,
      resultCount: results.length,
      page: window.location.pathname,
      timestamp: new Date().toISOString(),
    });
  }

  sendEvent(eventType, data) {
    const event = {
      eventType,
      ...this.baseData,
      ...data,
    };

    fetch("/api/track", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(event),
    }).catch((err) => console.error("트래킹 이벤트 전송 실패:", err));
  }
}

// 사용 예시
const tracker = new ActivityTracker(currentUser.id, generateSessionId());

// 페이지 로드 시
tracker.trackPageView(window.location.pathname);

// 버튼 클릭 시
document.getElementById("addToCart").addEventListener("click", function () {
  tracker.trackClick("addToCart", "button");
});

// 검색 시
document.getElementById("searchForm").addEventListener("submit", function (e) {
  e.preventDefault();
  const query = document.getElementById("searchInput").value;
  // 검색 수행...
  const results = performSearch(query);
  tracker.trackSearch(query, results);
});
```

#### 6.3.2 활동 이벤트 프로듀서 (API 엔드포인트)

```java
@RestController
public class TrackingController {

    @Autowired
    private KafkaTemplate<String, UserActivityEvent> kafkaTemplate;

    @PostMapping("/api/track")
    public ResponseEntity<Void> trackEvent(@RequestBody UserActivityEvent event) {
        // 이벤트 유효성 검증
        if (event.getUserId() == null || event.getEventType() == null) {
            return ResponseEntity.badRequest().build();
        }

        // 추가 메타데이터 설정
        event.setServerTimestamp(LocalDateTime.now());
        event.setSourceIp(getClientIp());

        // 이벤트 유형에 따른 토픽 선택
        String topic;
        switch (event.getEventType()) {
            case "PAGE_VIEW":
                topic = "user-pageviews";
                break;
            case "CLICK":
                topic = "user-clicks";
                break;
            case "SEARCH":
                topic = "user-searches";
                break;
            default:
                topic = "user-activities";
        }

        // Kafka로 이벤트 전송
        kafkaTemplate.send(topic, event.getUserId(), event);

        return ResponseEntity.ok().build();
    }

    private String getClientIp(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
            return xForwardedFor.split(",")[0];
        }
        return request.getRemoteAddr();
    }
}
```

#### 6.3.3 실시간 분석 (Kafka Streams)

```java
@Service
public class RealTimeAnalyticsService {

    @Autowired
    private StreamsBuilder streamsBuilder;

    @PostConstruct
    public void processPageViews() {
        // 페이지뷰 스트림 생성
        KStream<String, UserActivityEvent> pageViews = streamsBuilder.stream(
            "user-pageviews",
            Consumed.with(Serdes.String(), UserActivityEventSerde.serdes())
        );

        // 1분 윈도우로 페이지별 집계
        KTable<Windowed<String>, Long> pageViewCounts = pageViews
            .groupBy((key, event) -> event.getPage(),
                    Grouped.with(Serdes.String(), UserActivityEventSerde.serdes()))
            .windowedBy(TimeWindows.of(Duration.ofMinutes(1)))
            .count();

        // 결과를 토픽으로 내보내기
        pageViewCounts.toStream()
            .map((windowedPage, count) -> KeyValue.pair(
                windowedPage.key(),
                new PageViewStat(
                    windowedPage.key(),
                    windowedPage.window().start(),
                    windowedPage.window().end(),
                    count
                )
            ))
            .to("page-view-stats",
                Produced.with(Serdes.String(), PageViewStatSerde.serdes()));

        // 세션별 페이지 시퀀스 분석
        KTable<String, List<String>> userSessions = pageViews
            .groupBy((key, event) -> event.getSessionId(),
                    Grouped.with(Serdes.String(), UserActivityEventSerde.serdes()))
            .aggregate(
                ArrayList::new,
                (sessionId, event, pages) -> {
                    pages.add(event.getPage());
                    return pages;
                },
                Materialized.with(Serdes.String(), ListSerde.serdes(Serdes.String()))
            );

        // 세션 데이터 저장
        userSessions.toStream()
            .to("user-session-paths",
                Produced.with(Serdes.String(), ListSerde.serdes(Serdes.String())));
    }

    @PostConstruct
    public void processSearchEvents() {
        // 검색 이벤트 스트림
        KStream<String, UserActivityEvent> searches = streamsBuilder.stream(
            "user-searches",
            Consumed.with(Serdes.String(), UserActivityEventSerde.serdes())
        );

        // 인기 검색어 집계 (10분 윈도우)
        KTable<Windowed<String>, Long> popularSearches = searches
            .map((key, event) -> KeyValue.pair(
                event.getQuery().toLowerCase(),
                event
            ))
            .groupByKey(Grouped.with(Serdes.String(), UserActivityEventSerde.serdes()))
            .windowedBy(TimeWindows.of(Duration.ofMinutes(10)))
            .count();

        // 상위 10개 검색어 추출
        popularSearches.toStream()
            .map((windowedQuery, count) -> KeyValue.pair(
                windowedQuery.window().start() + "-" + windowedQuery.window().end(),
                new SearchTrend(
                    windowedQuery.key(),
                    count,
                    windowedQuery.window().start(),
                    windowedQuery.window().end()
                )
            ))
            .to("popular-searches",
                Produced.with(Serdes.String(), SearchTrendSerde.serdes()));
    }
}
```

#### 6.3.4 대시보드 서비스 (WebSocket으로 실시간 데이터 전송)

```java
@Service
public class AnalyticsDashboardService {

    @Autowired
    private SimpMessagingTemplate webSocketTemplate;

    @KafkaListener(topics = "page-view-stats", groupId = "dashboard-service")
    public void handlePageViewStats(PageViewStat stat) {
        // WebSocket을 통해 클라이언트에 데이터 전송
        webSocketTemplate.convertAndSend("/topic/pageviews", stat);
    }

    @KafkaListener(topics = "popular-searches", groupId = "dashboard-service")
    public void handlePopularSearches(SearchTrend trend) {
        // WebSocket을 통해 클라이언트에 데이터 전송
        webSocketTemplate.convertAndSend("/topic/searches", trend);
    }

    @Scheduled(fixedRate = 60000) // 1분마다 실행
    public void sendAggregatedStats() {
        // 최근 통계 집계 및 전송
        DashboardSummary summary = aggregateRecentStats();
        webSocketTemplate.convertAndSend("/topic/summary", summary);
    }

    private DashboardSummary aggregateRecentStats() {
        // 대시보드 요약 정보 집계 로직 구현
        return new DashboardSummary(
            getTotalActiveUsers(),
            getTotalPageViews(),
            getTopPages(),
            getTopSearches(),
            getConversionRate(),
            getCurrentTime()
        );
    }
}
```

## 7. Kafka 성능 관리 및 모니터링

### 7.1 모니터링 지표

Kafka 클러스터의 상태를 모니터링하기 위한 주요 지표들:

#### 7.1.1 브로커 지표

- **BytesInPerSec / BytesOutPerSec**: 유입/유출 데이터 처리량
- **RequestsPerSec**: 초당 요청 수
- **RequestQueueSize**: 처리 대기 중인 요청 수
- **UnderReplicatedPartitions**: 복제가 부족한 파티션 수
- **OfflinePartitionsCount**: 오프라인 상태인 파티션 수
- **ActiveControllerCount**: 활성 컨트롤러 수 (클러스터당 1개)

#### 7.1.2 토픽 지표

- **MessagesInPerSec**: 토픽별 초당 메시지 수
- **BytesInPerSec / BytesOutPerSec**: 토픽별 초당 데이터 처리량
- **FailedFetchRequestsPerSec**: 실패한 가져오기 요청 수
- **FailedProduceRequestsPerSec**: 실패한 생성 요청 수

#### 7.1.3 파티션 지표

- **LogStartOffset / LogEndOffset**: 시작 및 끝 오프셋
- **ReplicaLag**: 팔로워 복제본의 지연 시간
- **LastStableOffset**: 마지막 안정 오프셋 (트랜잭션)
- **Size**: 파티션 크기

#### 7.1.4 컨슈머 그룹 지표

- **ConsumerLag**: 컨슈머 지연 (프로듀서 오프셋과 컨슈머 오프셋 간 차이)
- **MaxLag**: 최대 지연량
- **MaxLagPartition**: 가장 큰 지연을 보이는 파티션
- **ConsumerAssignments**: 컨슈머별 할당된 파티션

### 7.2 모니터링 도구

#### 7.2.1 JMX 모니터링

Kafka는 JMX를 통해 메트릭을 노출합니다. 이를 활용하기 위한 도커 설정:

```yaml
environment:
  KAFKA_JMX_OPTS: "-Dcom.sun.management.jmxremote
    -Dcom.sun.management.jmxremote.port=9999
    -Dcom.sun.management.jmxremote.authenticate=false
    -Dcom.sun.management.jmxremote.ssl=false"
  JMX_PORT: 9999
ports:
  - "9999:9999"
```

#### 7.2.2 Prometheus + Grafana 설정

Prometheus 설정 파일 (prometheus.yml):

```yaml
scrape_configs:
  - job_name: "kafka"
    static_configs:
      - targets: ["kafka-exporter:9308"]

  - job_name: "zookeeper"
    static_configs:
      - targets: ["zookeeper-exporter:9141"]
```

Kafka Exporter 도커 구성:

```yaml
kafka-exporter:
  image: danielqsj/kafka-exporter
  container_name: kafka_exporter
  command:
    - "--kafka.server=kafka:9092"
    - "--topic.filter=.*"
    - "--group.filter=.*"
  ports:
    - "9308:9308"
  depends_on:
    - kafka
```

#### 7.2.3 Confluent Control Center

Confluent Platform의 GUI 모니터링 도구:

```yaml
control-center:
  image: confluentinc/cp-enterprise-control-center:7.4.0
  container_name: control_center
  depends_on:
    - kafka
    - zookeeper
  ports:
    - "9021:9021"
  environment:
    CONTROL_CENTER_BOOTSTRAP_SERVERS: "kafka:9092"
    CONTROL_CENTER_ZOOKEEPER_CONNECT: "zookeeper:2181"
    CONTROL_CENTER_REPLICATION_FACTOR: 1
    CONTROL_CENTER_MONITORING_INTERCEPTOR_TOPIC_REPLICATION: 1
    CONTROL_CENTER_INTERNAL_TOPICS_REPLICATION: 1
    CONTROL_CENTER_COMMAND_TOPIC_REPLICATION: 1
    CONTROL_CENTER_METRICS_TOPIC_REPLICATION: 1
    CONTROL_CENTER_STREAMS_NUM_STREAM_THREADS: 1
    CONTROL_CENTER_CONNECT_CLUSTER: "http://kafka-connect:8083"
```

### 7.3 성능 튜닝 전략

#### 7.3.1 프로듀서 튜닝

```java
// 프로듀서 속성 설정 예시
Properties props = new Properties();
props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class.getName());

// 성능 관련 설정
props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);  // 배치 크기 (바이트)
props.put(ProducerConfig.LINGER_MS_CONFIG, 5);       // 최대 지연 시간 (밀리초)
props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // 버퍼 메모리 (32MB)
props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy"); // 압축 타입
props.put(ProducerConfig.ACKS_CONFIG, "1");          // 응답 대기 설정 (0, 1, all)
props.put(ProducerConfig.RETRIES_CONFIG, 3);         // 재시도 횟수
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5); // 연결당 최대 요청 수

KafkaProducer<String, JsonNode> producer = new KafkaProducer<>(props);
```

#### 7.3.2 컨슈머 튜닝

```java
// 컨슈머 속성 설정 예시
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "order-processing-group");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class.getName());

// 성능 관련 설정
props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 1024); // 최소 가져오기 크기 (바이트)
props.put(ConsumerConfig.FETCH_MAX_BYTES_CONFIG, 52428800); // 최대 가져오기 크기 (50MB)
props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, 500); // 최대 대기 시간 (밀리초)
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500); // 폴링당 최대 레코드 수
props.put(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, 1048576); // 파티션당 최대 가져오기 크기 (1MB)
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest"); // 오프셋 리셋 전략
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false"); // 자동 커밋 비활성화
props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "5000"); // 자동 커밋 간격 (ms)

KafkaConsumer<String, JsonNode> consumer = new KafkaConsumer<>(props);
```

#### 7.3.3 토픽 설계 최적화

```bash
# 토픽 최적화 설정 예시
kafka-topics.sh --create \
  --bootstrap-server kafka:9092 \
  --topic high-throughput-topic \
  --partitions 16 \
  --replication-factor 3 \
  --config min.insync.replicas=2 \
  --config retention.ms=604800000 \
  --config cleanup.policy=delete \
  --config segment.bytes=1073741824 \
  --config compression.type=lz4
```

주요 토픽 설계 고려사항:

- **파티션 수**: 처리량과 병렬 처리 수준에 맞게 조정
- **복제 팩터**: 데이터 안정성과 가용성 요구사항 고려
- **보존 정책**: 데이터 저장 기간 및 스토리지 요구사항
- **세그먼트 크기**: 로그 관리 및 삭제 효율성에 영향
- **압축 타입**: CPU 사용량과 디스크 저장 공간 간 균형

## 8. 문제 해결 및 디버깅 가이드

### 8.1 일반적인 문제와 해결책

#### 8.1.1 브로커 연결 문제

- **증상**: 클라이언트가 브로커에 연결할 수 없음
- **해결책**:
  - 네트워크 연결 확인 (`ping`, `telnet`)
  - 리스너 설정 확인 (`KAFKA_LISTENERS`, `KAFKA_ADVERTISED_LISTENERS`)
  - 포트 개방 여부 확인 (방화벽 설정)
  - Docker 네트워크 설정 확인

#### 8.1.2 파티션 불균형

- **증상**: 특정 브로커에 파티션이 편중됨
- **해결책**:
  - 파티션 재분배 도구 사용
  ```bash
  kafka-reassign-partitions.sh --bootstrap-server kafka:9092 \
    --topics-to-move-json-file topics.json \
    --broker-list "1,2,3" \
    --generate
  ```
  - 자동 리밸런싱 설정 검토

#### 8.1.3 컨슈머 지연 (Consumer Lag)

- **증상**: 컨슈머가 프로듀서를 따라가지 못함
- **해결책**:
  - 컨슈머 성능 최적화 (배치 크기, 스레드 수)
  - 파티션 수 증가로 병렬 처리 강화
  - 컨슈머 인스턴스 추가
  - 처리 로직 최적화

#### 8.1.4 디스크 공간 부족

- **증상**: 브로커가 디스크 공간 부족으로 실패
- **해결책**:
  - 로그 보존 기간 감소
  - 불필요한 토픽 정리
  - 디스크 공간 확장
  - 압축 설정 활성화

### 8.2 디버깅 도구 및 명령

#### 8.2.1 토픽 정보 확인

```bash
# 토픽 목록 조회
docker exec -it kafka_prod kafka-topics.sh --bootstrap-server localhost:9092 --list

# 토픽 상세 정보 조회
docker exec -it kafka_prod kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic my-topic

# 특정 토픽의 메시지 확인
docker exec -it kafka_prod kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic my-topic --from-beginning --max-messages 10
```

#### 8.2.2 컨슈머 그룹 정보 확인

```bash
# 컨슈머 그룹 목록 조회
docker exec -it kafka_prod kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 컨슈머 그룹 상세 정보 (LAG 포함)
docker exec -it kafka_prod kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-consumer-group

# 컨슈머 그룹 오프셋 리셋
docker exec -it kafka_prod kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-consumer-group --topic my-topic --reset-offsets --to-earliest --execute
```

#### 8.2.3 브로커 상태 확인

```bash
# 클러스터 정보 확인
docker exec -it kafka_prod kafka-broker-api-versions.sh --bootstrap-server localhost:9092

# 로그 확인
docker logs kafka_prod

# 설정 확인
docker exec -it kafka_prod cat /etc/kafka/server.properties
```

#### 8.2.4 ZooKeeper 정보 확인

```bash
# ZooKeeper 연결
docker exec -it zookeeper_prod zkCli.sh -server localhost:2181

# Kafka 브로커 정보 확인
ls /brokers/ids

# 토픽 정보 확인
ls /brokers/topics

# 컨트롤러 정보 확인
get /controller
```

### 8.3 재해 복구 전략

#### 8.3.1 백업 및 복원

- **로그 세그먼트 백업**: 파티션 로그 디렉토리 정기 백업
- **설정 파일 백업**: 브로커 및 토픽 설정 백업
- **MirrorMaker 활용**: 다른 클러스터로 데이터 미러링

#### 8.3.2 장애 복구 절차

1. **브로커 장애**:
   - 새 브로커 시작 (동일한 ID)
   - 자동 파티션 리더 선출 및 동기화 확인
2. **전체 클러스터 장애**:
   - ZooKeeper 앙상블 복구 및 시작
   - 브로커 순차적 시작
   - 토픽 및 파티션 상태 확인
3. **데이터 손상**:
   - 백업에서 로그 세그먼트 복원
   - 필요 시 복제본에서 데이터 복구
   - 컨슈머 오프셋 조정

## 9. 참고 자료 및 추가 학습 리소스

### 9.1 공식 문서

- [Apache Kafka 공식 문서](https://kafka.apache.org/documentation/)
- [Apache ZooKeeper 공식 문서](https://zookeeper.apache.org/doc/current/)
- [Confluent Platform 문서](https://docs.confluent.io/)
- [Kafka Docker 이미지](https://hub.docker.com/r/confluentinc/cp-kafka/)

### 9.2 책 추천

- "Kafka: The Definitive Guide" by Gwen Shapira, Neha Narkhede, and Todd Palino
- "Kafka Streams in Action" by Bill Bejeck
- "Mastering Apache Kafka" by Nishant Garg

### 9.3 온라인 강좌 및 튜토리얼

- [Confluent Kafka 튜토리얼](https://developer.confluent.io/tutorials/)
- [Apache Kafka Series - Udemy 강좌](https://www.udemy.com/course/apache-kafka/)
- [Kafka for Beginners - Conduktor 아카데미](https://www.conduktor.io/kafka/)# Kafka & Zookeeper 플랫폼 상세 문서

## 1. Apache Kafka 개요

Apache Kafka는 분산 이벤트 스트리밍 플랫폼으로, 고성능, 고가용성, 확장성을 제공하며 실시간 데이터 파이프라인 구축과 스트리밍 애플리케이션 개발에 사용됩니다. LinkedIn에서 처음 개발되어 현재는 Apache Software Foundation의 오픈 소스 프로젝트로 관리되고 있습니다.

### 1.1 핵심 특징

- **높은 처리량**: 초당 수백만 개의 메시지 처리 가능
- **낮은 지연 시간**: 밀리초 단위의 지연 시간으로 데이터 전달
- **내구성**: 디스크에 데이터 저장으로 안정성 보장
- **확장성**: 클러스터 노드 추가로 수평적 확장 가능
- **내결함성**: 브로커 장애 시에도 데이터 손실 없이 작동
- **분산 아키텍처**: 다수 서버에 걸쳐 데이터 분산 저장 및 처리
- **스트림 처리**: Kafka Streams API를 통한 실시간 데이터 처리

### 1.2 주요 활용 사례

- **메시지 큐잉**: 시스템 간 비동기 메시지 전달
- **로그 집계**: 여러 시스템의 로그 수집 및 중앙화
- **스트림 처리**: 실시간 데이터 변환 및 분석
- **이벤트 소싱**: 상태 변경을 이벤트로 저장하는 패턴
- **활동 추적**: 사용자 행동 및 시스템 활동 모니터링
- **메트릭 수집**: 시스템 모니터링 데이터 수집
- **마이크로서비스 통신**: 서비스 간 비동기 통신 구현

## 2. Zookeeper 개요

Apache ZooKeeper는 분산 시스템을 위한 코디네이션 서비스로, 설정 관리, 네이밍, 동기화, 그룹 서비스 등을 제공합니다. Kafka는 클러스터 상태 관리와 브로커 조정을 위해 ZooKeeper를 사용합니다.

### 2.1 핵심 특징

- **간단한 분산 코디네이션**: 높은 신뢰성을 가진 분산 코디네이션 서비스
- **순차적 일관성**: 클라이언트 요청을 순서대로 처리
- **원자성**: 요청이 성공하거나 실패하거나 중간 상태 없음
- **단일 시스템 이미지**: 서버가 다수여도 클라이언트는 단일 서비스처럼 접근
- **신뢰성**: 다수의 서버가 실패해도 서비스 유지
- **적시성**: 시스템 상태 갱신 시 클라이언트에 빠르게 통지

### 2.2 Kafka에서의 Zookeeper 역할

- **브로커 등록 관리**: Kafka 브로커 등록 및 상태 추적
- **토픽 설정 저장**: 토픽 구성 정보 저장
- **컨트롤러 선출**: 클러스터 컨트롤러 브로커 선정
- **ACL 관리**: 접근 제어 목록 저장
- **파티션 리더 추적**: 각 파티션 리더 정보 관리
- **Kafka 버전 3.0 이후**: Zookeeper 의존성 제거 계획(KIP-500) 진행 중

## 3. Kafka 아키텍처 및 동작 원리

### 3.1 핵심 개념

#### 3.1.1 브로커(Broker)

Kafka 서버로, 메시지를 저장하고 전달하는 역할을 합니다. 클러스터는 여러 브로커로 구성되며, 각 브로커는 고유한 ID를 가집니다.

#### 3.1.2 토픽(Topic)

메시지를 카테고리별로 구분하는 논리적 단위입니다. 각 토픽은 여러 파티션으로 구성될 수 있습니다.

#### 3.1.3 파티션(Partition)

토픽을 여러 부분으로 나눈 것으로, 병렬 처리와 확장성을 제공합니다. 각 파티션은 순서가 보장된 메시지 시퀀스입니다.

#### 3.1.4 복제(Replication)

데이터 손실 방지를 위해 파티션을 여러 브로커에 복제합니다. 복제 팩터는 각 파티션이 몇 개의 복제본을 가질지 지정합니다.

#### 3.1.5 프로듀서(Producer)

메시지를 생성하여 토픽에 게시하는 클라이언트 애플리케이션입니다.

#### 3.1.6 컨슈머(Consumer)

토픽에서 메시지를 구독하고 처리하는 클라이언트 애플리케이션입니다.

#### 3.1.7 컨슈머 그룹(Consumer Group)

하나의 토픽을 공동으로 소비하는 컨슈머 집합입니다. 각 파티션은 그룹 내의 하나의 컨슈머만 소비할 수 있습니다.

#### 3.1.8 오프셋(Offset)

파티션 내 각 메시지의 위치를 나타내는 순차적 ID입니다. 컨슈머는 오프셋을 통해 소비 위치를 추적합니다.

### 3.2 데이터 흐름

1. **프로듀서**가 메시지를 특정 토픽에 게시합니다.
2. 메시지는 해당 토픽의 특정 **파티션**에 기록됩니다.
3. 파티션은 지정된 복제 팩터에 따라 여러 **브로커**에 복제됩니다.
4. 각 파티션에는 리더와 팔로워가 있으며, 모든 읽기/쓰기 요청은 **리더**를 통해 이루어집니다.
5. **컨슈머**는 특정 파티션에서 오프셋을 기준으로 메시지를 가져옵니다.
6. 컨슈머가 그룹의 일부인 경우, 그룹 내 컨슈머 간에 파티션이 균등하게 분배됩니다.

![Kafka Architecture](https://example.com/kafka-architecture.png)

### 3.3 메시지 전달 보장

Kafka는 세 가지 수준의 메시지 전달 보장을 제공합니다:

1. **At most once**: 메시지가 최대 한 번 전달되며, 손실될 수 있습니다.
2. **At least once**: 메시지가 최소 한 번 전달되며, 중복될 수 있습니다.
3. **Exactly once**: 메시지가 정확히 한 번 전달됩니다 (Kafka Streams API나 트랜잭션 API 사용 시).

### 3.4 데이터 보존 정책

Kafka는 다양한 데이터 보존 정책을 지원합니다:

- **시간 기반**: 설정된 시간(예: 7일) 이후 메시지 삭제
- **크기 기반**: 특정 크기에 도달하면 오래된 메시지부터 삭제
- **압축(Compacted)**: 각 키의 최신 값만 유지하고 이전 값은 삭제

## 4. Kafka와 Zookeeper 도커 구성 분석

현재 프로젝트에서 사용 중인 Kafka와 Zookeeper 도커 구성을 살펴보겠습니다:

### 4.1 Zookeeper Dockerfile 분석

```dockerfile
FROM confluentinc/cp-zookeeper:7.4.0

ENV ZOOKEEPER_CLIENT_PORT=2181
ENV ZOOKEEPER_TICK_TIME=2000

EXPOSE 2181

CMD ["/etc/confluent/docker/run"]
```

구성 요소:

- **베이스 이미지**: Confluent Platform의 Zookeeper 7.4.0 이미지 사용
- **환경 변수**:
  - `ZOOKEEPER_CLIENT_PORT`: Zookeeper 클라이언트 포트 설정 (2181)
  - `ZOOKEEPER_TICK_TIME`: 기본 시간 단위 설정 (밀리초 단위, 2000ms = 2초)
- **포트 노출**: 클라이언트 포트 2181 노출
- **실행 명령**: Confluent 제공 스크립트로 Zookeeper 실행
