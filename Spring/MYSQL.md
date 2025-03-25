# MySQL 플랫폼 상세 문서

## 1. MySQL 개요

MySQL은 세계에서 가장 널리 사용되는 오픈 소스 관계형 데이터베이스 관리 시스템(RDBMS)입니다. 현재 Oracle Corporation이 소유하고 있으며, 다양한 웹 애플리케이션 및 온라인 서비스의 백엔드 데이터 저장소로 널리 사용됩니다.

### 1.1 핵심 특징

- **관계형 데이터베이스**: 테이블, 열 및 행으로 구성된 구조화된 데이터 저장
- **ACID 준수**: 트랜잭션의 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability) 보장
- **다양한 스토리지 엔진**: InnoDB(기본), MyISAM, Memory 등 다양한 스토리지 엔진 지원
- **SQL 지원**: 표준 SQL 문법 지원으로 데이터 관리 가능
- **확장성**: 대규모 데이터 세트와 높은 트래픽을 처리할 수 있는 능력
- **복제 및 파티셔닝**: 고가용성 및 성능 최적화를 위한 기능 제공

## 2. 아키텍처 및 동작 원리

### 2.1 MySQL 아키텍처

MySQL의 아키텍처는 크게 다음과 같은 구성 요소로 이루어져 있습니다:

![MySQL Architecture](https://example.com/mysql-architecture.png)

1. **클라이언트 레이어**:

   - 애플리케이션에서 MySQL 서버에 연결하는 클라이언트 프로그램

2. **서버 레이어**:

   - **연결 처리**: 클라이언트 연결 관리
   - **쿼리 캐시**: 동일한 쿼리 결과를 캐싱 (MySQL 8.0에서 제거됨)
   - **파서**: SQL 구문 분석
   - **옵티마이저**: 쿼리 실행 최적화
   - **실행기**: 실제 쿼리 실행

3. **스토리지 엔진 레이어**:

   - 데이터 저장, 검색 및 업데이트 처리
   - 다양한 스토리지 엔진 지원 (InnoDB, MyISAM 등)

4. **파일 시스템 레이어**:
   - 물리적 데이터 저장

### 2.2 주요 스토리지 엔진

#### 2.2.1 InnoDB (기본 스토리지 엔진)

현재 MySQL의 기본 스토리지 엔진으로, 다음과 같은 특징을 가집니다:

- **트랜잭션 지원**: ACID 속성 준수
- **외래 키 제약 조건**: 참조 무결성 보장
- **행 수준 잠금**: 높은 동시성 지원
- **자동 충돌 복구**: 시스템 장애 후 자동 복구
- **버퍼 풀**: 데이터와 인덱스를 메모리에 캐싱

내부 구조:

- **시스템 테이블스페이스**: 데이터 딕셔너리, undo 로그 저장
- **개별 테이블스페이스**: 각 테이블별 데이터 파일
- **redo 로그**: 장애 복구를 위한 변경 기록
- **undo 로그**: 트랜잭션 롤백을 위한 기록
- **변경 버퍼**: 보조 인덱스 업데이트 최적화
- **더블 라이트 버퍼**: 부분 페이지 쓰기 방지

#### 2.2.2 MyISAM

이전 버전의 MySQL에서 기본 스토리지 엔진으로 사용되었습니다:

- **비-트랜잭션**: 트랜잭션을 지원하지 않음
- **테이블 수준 잠금**: 제한된 동시성
- **전체 텍스트 검색**: 텍스트 검색에 최적화
- **디스크 공간 효율**: 데이터 압축 기능

### 2.3 쿼리 처리 과정

MySQL에서 쿼리가 처리되는 과정은 다음과 같습니다:

1. **연결 설정**: 클라이언트가 MySQL 서버에 연결
2. **쿼리 전송**: 클라이언트가 SQL 쿼리를 서버로 전송
3. **구문 분석**: 서버가 SQL 구문 분석 및 유효성 검사
4. **최적화**: 쿼리 실행 계획 생성 및 최적화
5. **실행**: 최적화된 쿼리 실행 계획에 따라 쿼리 실행
6. **결과 반환**: 실행 결과를 클라이언트에 반환

### 2.4 트랜잭션 처리

InnoDB 스토리지 엔진에서 트랜잭션은 다음과 같이 처리됩니다:

1. **트랜잭션 시작**: `START TRANSACTION` 또는 `BEGIN` 명령
2. **데이터 수정**: `INSERT`, `UPDATE`, `DELETE` 등의 작업 수행
3. **커밋 또는 롤백**: `COMMIT` 또는 `ROLLBACK` 명령으로 트랜잭션 완료

트랜잭션 격리 수준:

- **READ UNCOMMITTED**: 커밋되지 않은 데이터 읽기 가능
- **READ COMMITTED**: 커밋된 데이터만 읽기 가능
- **REPEATABLE READ** (기본값): 트랜잭션 내에서 일관된 읽기 보장
- **SERIALIZABLE**: 완전한 격리, 가장 엄격한 수준

## 3. MySQL 도커 구성 분석

현재 프로젝트에서 사용 중인 MySQL 도커 구성을 살펴보겠습니다:

### 3.1 Dockerfile 분석

```dockerfile
FROM mysql:8.0

ENV MYSQL_ROOT_PASSWORD=rootpass
ENV MYSQL_DATABASE=freamdb
ENV MYSQL_USER=fream
ENV MYSQL_PASSWORD=fream

EXPOSE 3306

# 만약 my.cnf 필요시:
# COPY my.cnf /etc/mysql/conf.d/my.cnf

CMD ["mysqld"]
```

구성 요소:

- **베이스 이미지**: MySQL 8.0 공식 이미지 사용
- **환경 변수**:
  - `MYSQL_ROOT_PASSWORD`: MySQL root 사용자 비밀번호
  - `MYSQL_DATABASE`: 기본 생성할 데이터베이스 이름
  - `MYSQL_USER`: 추가 사용자 계정
  - `MYSQL_PASSWORD`: 추가 사용자 비밀번호
- **포트**: 기본 MySQL 포트인 3306 노출
- **실행 명령**: `mysqld` 서버 프로세스 실행

### 3.2 docker-compose.yml 설정

```yaml
services:
  mysql:
    build:
      context: ../..
      dockerfile: docker/mysql/Dockerfile-mysql
    container_name: mysql_prod
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: freamdb
      MYSQL_USER: fream
      MYSQL_PASSWORD: fream
      TZ: Asia/Seoul
    ports:
      - "3306:3306"
    volumes:
      - mysql_data_prod:/var/lib/mysql
```

주요 설정:

- **볼륨 마운트**: `mysql_data_prod` 볼륨을 `/var/lib/mysql`에 마운트하여 데이터 영속성 보장
- **시간대 설정**: 한국 시간대(Asia/Seoul) 사용
- **포트 매핑**: 호스트의 3306 포트를 컨테이너의 3306 포트에 매핑

## 4. MySQL 최적화 및 운영 가이드

### 4.1 성능 최적화 전략

#### 4.1.1 하드웨어 최적화

- **메모리**: InnoDB 버퍼 풀 크기를 적절히 설정 (가용 메모리의 50-80%)
- **디스크**: 빠른 SSD 스토리지 사용으로 I/O 성능 향상
- **CPU**: 멀티 코어 활용을 위한 스레드 설정 최적화

#### 4.1.2 구성 파일 최적화 (my.cnf)

```ini
[mysqld]
# InnoDB 설정
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT

# 쿼리 캐시 설정 (MySQL 8.0 이전 버전용)
# query_cache_type = 1
# query_cache_size = 64M

# 연결 및 스레드 설정
max_connections = 500
thread_cache_size = 128

# 임시 테이블 설정
tmp_table_size = 64M
max_heap_table_size = 64M
```

#### 4.1.3 인덱스 최적화

- **적절한 인덱스 설계**: 자주 사용되는 WHERE, JOIN, ORDER BY 절의 컬럼에 인덱스 생성
- **복합 인덱스**: 여러 컬럼을 사용하는 쿼리를 위한 복합 인덱스 설계
- **인덱스 모니터링**: 불필요하거나 사용되지 않는 인덱스 제거

#### 4.1.4 쿼리 최적화

- **EXPLAIN 활용**: 쿼리 실행 계획 분석으로 병목 지점 식별
- **느린 쿼리 로그**: 성능 문제가 있는 쿼리 식별 및 최적화
- **쿼리 재작성**: 서브쿼리보다 JOIN 사용, 불필요한 컬럼 제거

### 4.2 백업 및 복구 전략

#### 4.2.1 mysqldump를 이용한 논리적 백업

```bash
# 전체 데이터베이스 백업
mysqldump -u root -p --all-databases > full_backup.sql

# 특정 데이터베이스 백업
mysqldump -u root -p freamdb > freamdb_backup.sql

# 압축 백업
mysqldump -u root -p freamdb | gzip > freamdb_backup.sql.gz
```

#### 4.2.2 물리적 백업 (XtraBackup)

```bash
# 전체 백업
xtrabackup --backup --target-dir=/backup/full

# 증분 백업
xtrabackup --backup --target-dir=/backup/inc1 \
  --incremental-basedir=/backup/full
```

#### 4.2.3 자동 백업 스크립트 예시

```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/mysql"

# 백업 디렉토리 생성
mkdir -p $BACKUP_DIR

# mysqldump 실행
docker exec mysql_prod mysqldump -u root -prootpass freamdb | gzip > $BACKUP_DIR/freamdb_$DATE.sql.gz

# 30일 이상된 백업 파일 제거
find $BACKUP_DIR -type f -name "*.gz" -mtime +30 -delete
```

### 4.3 모니터링 및 유지보수

#### 4.3.1 주요 모니터링 지표

- **연결 수**: 활성 및 최대 연결 수
- **쿼리 처리량**: 초당 쿼리 수
- **테이블 크기**: 데이터 및 인덱스 크기
- **버퍼 풀 사용률**: 메모리 사용 상태
- **디스크 I/O**: 읽기/쓰기 작업량

#### 4.3.2 모니터링 도구

- **MySQL Enterprise Monitor**: 상업용 모니터링 도구
- **Prometheus + Grafana**: 오픈 소스 모니터링 솔루션
- **Percona Monitoring and Management (PMM)**: 무료 모니터링 도구

#### 4.3.3 정기 유지보수 작업

- **테이블 최적화**: `OPTIMIZE TABLE` 명령으로 단편화 해결
- **통계 업데이트**: `ANALYZE TABLE` 명령으로 인덱스 통계 갱신
- **로그 순환**: 로그 파일 관리로 디스크 공간 확보

## 5. MySQL 활용 사례 및 응용

### 5.1 웹 애플리케이션에서의 활용

#### 5.1.1 사용자 인증 및 권한 관리

```sql
-- 사용자 테이블 예시
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 권한 관리 테이블 예시
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

#### 5.1.2 세션 관리

```sql
CREATE TABLE sessions (
    id VARCHAR(128) PRIMARY KEY,
    user_id INT,
    ip_address VARCHAR(45),
    user_agent TEXT,
    payload TEXT,
    last_activity INT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

#### 5.1.3 트랜잭션을 활용한 결제 처리

```sql
START TRANSACTION;

-- 주문 생성
INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 150.00, 'pending');
SET @order_id = LAST_INSERT_ID();

-- 주문 상세 항목 추가
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 101, 2, 75.00);

-- 재고 감소
UPDATE products SET stock = stock - 2 WHERE id = 101;

-- 사용자 포인트 사용
UPDATE users SET points = points - 50 WHERE id = 1;

COMMIT;
```

### 5.2 데이터 분석 및 보고서 생성

#### 5.2.1 데이터 집계 쿼리

```sql
-- 일별 판매 요약
SELECT
    DATE(created_at) AS sale_date,
    COUNT(*) AS order_count,
    SUM(total_amount) AS daily_revenue
FROM
    orders
WHERE
    status = 'completed'
    AND created_at >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY
    DATE(created_at)
ORDER BY
    sale_date DESC;
```

#### 5.2.2 뷰를 활용한 데이터 추상화

```sql
-- 제품 판매 분석을 위한 뷰
CREATE VIEW product_sales_analysis AS
SELECT
    p.id AS product_id,
    p.name AS product_name,
    p.category,
    COUNT(oi.id) AS times_ordered,
    SUM(oi.quantity) AS total_quantity,
    SUM(oi.quantity * oi.price) AS total_revenue
FROM
    products p
LEFT JOIN
    order_items oi ON p.id = oi.product_id
LEFT JOIN
    orders o ON oi.order_id = o.id
WHERE
    o.status = 'completed'
GROUP BY
    p.id, p.name, p.category;
```

#### 5.2.3 저장 프로시저를 활용한 보고서 생성

```sql
DELIMITER //
CREATE PROCEDURE generate_monthly_report(IN year_num INT, IN month_num INT)
BEGIN
    DECLARE start_date DATE;
    DECLARE end_date DATE;

    SET start_date = DATE(CONCAT(year_num, '-', month_num, '-01'));
    SET end_date = LAST_DAY(start_date);

    -- 월별 판매 요약
    SELECT
        COUNT(*) AS total_orders,
        SUM(total_amount) AS total_revenue,
        AVG(total_amount) AS avg_order_value
    FROM
        orders
    WHERE
        status = 'completed'
        AND created_at BETWEEN start_date AND end_date;

    -- 카테고리별 판매 분석
    SELECT
        p.category,
        SUM(oi.quantity) AS items_sold,
        SUM(oi.quantity * oi.price) AS category_revenue,
        ROUND(SUM(oi.quantity * oi.price) /
            (SELECT SUM(total_amount) FROM orders
             WHERE status = 'completed'
             AND created_at BETWEEN start_date AND end_date) * 100, 2) AS revenue_percentage
    FROM
        products p
    JOIN
        order_items oi ON p.id = oi.product_id
    JOIN
        orders o ON oi.order_id = o.id
    WHERE
        o.status = 'completed'
        AND o.created_at BETWEEN start_date AND end_date
    GROUP BY
        p.category
    ORDER BY
        category_revenue DESC;
END //
DELIMITER ;
```

### 5.3 MySQL과 다른 시스템 연동

#### 5.3.1 Spring Boot와 연동

```java
// application.properties 설정
spring.datasource.url=jdbc:mysql://mysql:3306/freamdb
spring.datasource.username=fream
spring.datasource.password=fream
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

// 엔티티 클래스 예시
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private BigDecimal price;

    @Column
    private String description;

    // getters and setters
}

// 리포지토리 인터페이스 예시
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategory(String category);
    List<Product> findByPriceBetween(BigDecimal min, BigDecimal max);

    @Query("SELECT p FROM Product p WHERE p.name LIKE %:keyword% OR p.description LIKE %:keyword%")
    List<Product> searchProducts(@Param("keyword") String keyword);
}
```

#### 5.3.2 Elasticsearch와 데이터 동기화

```java
// MySQL의 제품 데이터를 Elasticsearch로 동기화하는 서비스 예시
@Service
public class ProductSyncService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ElasticsearchOperations elasticsearchOperations;

    @Scheduled(fixedRate = 3600000) // 1시간마다 실행
    public void syncProductsToElasticsearch() {
        List<Product> products = productRepository.findAll();

        List<ProductDocument> productDocuments = products.stream()
            .map(this::convertToDocument)
            .collect(Collectors.toList());

        elasticsearchOperations.bulkIndex(
            productDocuments.stream()
                .map(product -> new IndexQuery.Builder()
                    .withId(product.getId().toString())
                    .withObject(product)
                    .build())
                .collect(Collectors.toList()),
            IndexCoordinates.of("products")
        );
    }

    private ProductDocument convertToDocument(Product product) {
        // Product 엔티티를 Elasticsearch 문서로 변환
        return ProductDocument.builder()
            .id(product.getId())
            .name(product.getName())
            .description(product.getDescription())
            .price(product.getPrice())
            .category(product.getCategory())
            .build();
    }
}
```

#### 5.3.3 Kafka와 연동한 이벤트 기반 아키텍처

```java
// 주문 생성 후 Kafka로 이벤트 발행
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    @Transactional
    public Order createOrder(OrderRequest request) {
        // 주문 생성 로직
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setTotalAmount(calculateTotal(request.getItems()));
        order.setStatus("pending");

        // 주문 저장
        Order savedOrder = orderRepository.save(order);

        // 주문 상세 항목 저장
        saveOrderItems(savedOrder, request.getItems());

        // Kafka 이벤트 발행
        OrderCreatedEvent event = new OrderCreatedEvent(
            savedOrder.getId(),
            savedOrder.getUserId(),
            savedOrder.getTotalAmount(),
            request.getItems()
        );

        kafkaTemplate.send("order-events", event);

        return savedOrder;
    }

    // 기타 메소드...
}
```

## 6. 문제 해결 및 디버깅 가이드

### 6.1 일반적인 문제와 해결책

#### 6.1.1 연결 문제

- **최대 연결 수 초과**: `max_connections` 값 증가
- **접속 권한 문제**: 사용자 권한 및 호스트 설정 확인
- **네트워크 문제**: 방화벽, 포트 설정 확인

#### 6.1.2 성능 문제

- **느린 쿼리**: `EXPLAIN` 명령으로 실행 계획 분석 및 인덱스 최적화
- **메모리 부족**: 버퍼 풀 크기 조정 및 쿼리 최적화
- **디스크 I/O 병목**: SSD 사용, 인덱스 최적화, 불필요한 쿼리 제거

#### 6.1.3 데이터 불일치 문제

- **트랜잭션 격리 수준**: 애플리케이션 요구에 맞는 격리 수준 설정
- **외래 키 제약 조건**: 참조 무결성 유지를 위한 제약 조건 확인
- **잘못된 쿼리 로직**: 트랜잭션 범위 및 쿼리 로직 검토

### 6.2 로깅 및 모니터링

#### 6.2.1 로그 설정

```ini
[mysqld]
# 일반 쿼리 로그
general_log = 1
general_log_file = /var/log/mysql/query.log

# 느린 쿼리 로그
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
log_queries_not_using_indexes = 1

# 오류 로그
log_error = /var/log/mysql/error.log
```

#### 6.2.2 성능 스키마 활용

```sql
-- 가장 자주 실행되는 쿼리 확인
SELECT
    digest_text,
    count_star,
    avg_timer_wait
FROM
    performance_schema.events_statements_summary_by_digest
ORDER BY
    count_star DESC
LIMIT 10;

-- 테이블별 I/O 통계 확인
SELECT
    object_schema,
    object_name,
    count_read,
    count_write,
    count_fetch,
    count_insert,
    count_update,
    count_delete
FROM
    performance_schema.table_io_waits_summary_by_table
ORDER BY
    count_star DESC
LIMIT 10;
```

### 6.3 Docker 환경에서의 디버깅

#### 6.3.1 로그 확인

```bash
# 컨테이너 로그 확인
docker logs mysql_prod

# 실시간 로그 추적
docker logs -f mysql_prod
```

#### 6.3.2 컨테이너 내부 접속

```bash
# MySQL 컨테이너에 접속
docker exec -it mysql_prod bash

# MySQL 클라이언트 실행
mysql -u fream -p
```

#### 6.3.3 데이터베이스 상태 확인

```sql
-- 서버 상태 변수 확인
SHOW GLOBAL STATUS;

-- 프로세스 목록 확인
SHOW PROCESSLIST;

-- 실행 중인 트랜잭션 확인
SELECT * FROM information_schema.innodb_trx;

-- 테이블 상태 확인
SHOW TABLE STATUS;
```

## 7. 참고 자료 및 추가 학습 리소스

### 7.1 공식 문서

- [MySQL 서버 문서](https://dev.mysql.com/doc/refman/8.0/en/)
- [MySQL 퍼포먼스 튜닝](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
- [MySQL Docker 이미지](https://hub.docker.com/_/mysql)
