# Redis 플랫폼 상세 문서

## 1. Redis 개요

Redis(REmote DIctionary Server)는 인메모리 데이터 구조 저장소로, 데이터베이스, 캐시, 메시지 브로커 및 스트리밍 엔진으로 사용됩니다. 오픈 소스 프로젝트로, 빠른 읽기/쓰기 속도와 다양한 데이터 구조를 지원하는 것이 특징입니다.

### 1.1 핵심 특징

- **인메모리 데이터 저장**: 모든 데이터를 RAM에 저장하여 초고속 액세스 제공
- **다양한 데이터 구조**: 문자열, 해시, 리스트, 집합, 정렬된 집합, 비트맵, HyperLogLog 등
- **영속성**: 메모리에 있는 데이터를 디스크에 저장하는 옵션 제공 (RDB 및 AOF)
- **복제 및 클러스터링**: 고가용성 및 확장성을 위한 기능 지원
- **Lua 스크립팅**: 서버 측에서 스크립트 실행 가능
- **트랜잭션**: 여러 명령어를 원자적으로 실행 가능
- **게시/구독**: 메시징 시스템으로 활용 가능
- **만료 시간 설정**: 키에 TTL(Time To Live) 설정 가능

## 2. 아키텍처 및 동작 원리

### 2.1 Redis 아키텍처

Redis는 단일 스레드 아키텍처를 기본으로 하며 (Redis 6.0부터는 I/O 멀티스레딩 지원), 다음과 같은 구성요소로 이루어져 있습니다:

![Redis Architecture](https://example.com/redis-architecture.png)

1. **이벤트 루프**:

   - 단일 스레드 모델로 이벤트 기반 비동기 I/O 처리
   - 클라이언트 요청, 타이머 이벤트 등을 처리

2. **명령 처리기**:

   - 클라이언트 요청에 따른 명령 해석 및 실행
   - 데이터 구조 조작 및 응답 생성

3. **데이터 구조**:

   - 메모리 내 데이터 저장을 위한 다양한 데이터 구조
   - 키-값 맵핑을 위한 내부 해시 테이블

4. **영속성 메커니즘**:

   - RDB (Redis Database): 특정 시점의 스냅샷 저장
   - AOF (Append Only File): 모든 쓰기 명령을 로그에 기록

5. **복제 및 센티널**:
   - 마스터-슬레이브 복제를 통한 데이터 중복
   - 센티널을 통한 자동 장애 조치

### 2.2 데이터 구조 및 사용법

#### 2.2.1 문자열 (Strings)

가장 기본적인 데이터 타입으로, 텍스트, 직렬화된 객체, 정수 등을 저장할 수 있습니다.

```bash
# 문자열 설정 및 가져오기
SET user:1:name "John Doe"
GET user:1:name

# 숫자 증가/감소
SET counter 100
INCR counter      # 101
INCRBY counter 50 # 151
```

#### 2.2.2 해시 (Hashes)

필드-값 쌍의 컬렉션으로, 객체를 표현하는데 적합합니다.

```bash
# 해시 설정 및 가져오기
HSET user:1 name "John" email "john@example.com" age 30
HGET user:1 name  # "John"
HGETALL user:1    # 모든 필드와 값 반환
```

#### 2.2.3 리스트 (Lists)

순서가 있는 문자열 모음으로, 스택이나 큐로 사용 가능합니다.

```bash
# 리스트 추가 및 가져오기
LPUSH notifications "New message"
RPUSH notifications "Friend request"
LRANGE notifications 0 -1  # 모든 항목 조회
```

#### 2.2.4 집합 (Sets)

중복되지 않는 문자열의 비순서 컬렉션입니다.

```bash
# 집합 관리
SADD tags "redis" "database" "nosql"
SMEMBERS tags  # 모든 멤버 조회
SISMEMBER tags "redis"  # 멤버십 확인
```

#### 2.2.5 정렬된 집합 (Sorted Sets)

스코어와 연결된 문자열의 컬렉션으로, 항상 스코어 순으로 정렬됩니다.

```bash
# 정렬된 집합 관리
ZADD leaderboard 100 "player1" 80 "player2" 120 "player3"
ZRANGE leaderboard 0 -1  # 스코어 오름차순 조회
ZREVRANGE leaderboard 0 -1  # 스코어 내림차순 조회
```

### 2.3 영속성 메커니즘

Redis는 인메모리 데이터베이스이지만, 다음과 같은 방식으로 데이터 영속성을 제공합니다:

#### 2.3.1 RDB (Redis Database) 스냅샷

- 특정 시점의 데이터를 디스크에 저장하는 방식
- 설정된 시간 간격이나 특정 변경 수에 따라 스냅샷 생성
- 빠른 재시작과 백업에 적합

```conf
# redis.conf 설정 예시
save 900 1    # 900초(15분) 동안 1개 이상의 키가 변경되면 스냅샷 생성
save 300 10   # 300초(5분) 동안 10개 이상의 키가 변경되면 스냅샷 생성
save 60 10000 # 60초(1분) 동안 10000개 이상의 키가 변경되면 스냅샷 생성
```

#### 2.3.2 AOF (Append Only File)

- 모든 쓰기 명령을 로그 파일에 순차적으로 기록
- 서버 재시작 시 명령을 다시 실행하여 데이터 복구
- 더 안정적이지만 파일 크기가 커지고 처리 속도가 느림

```conf
# redis.conf 설정 예시
appendonly yes
appendfsync everysec  # 1초마다 AOF 파일 동기화 (always, everysec, no 중 선택)
```

#### 2.3.3 혼합 방식

- RDB와 AOF를 함께 사용하여 장점 활용
- Redis 4.0부터 AOF 파일에 RDB 프리앰블을 포함하는 혼합 방식 지원

```conf
# redis.conf 설정 예시
appendonly yes
aof-use-rdb-preamble yes
```

### 2.4 복제 및 고가용성

#### 2.4.1 마스터-슬레이브 복제

- 마스터 노드의 데이터를 하나 이상의 슬레이브 노드에 복제
- 읽기 확장성 및 데이터 중복성 제공

```conf
# 슬레이브 설정 예시 (redis.conf)
replicaof 192.168.1.100 6379  # 마스터 노드 IP와 포트
```

#### 2.4.2 Redis Sentinel

- 고가용성을 위한 분산 시스템
- 마스터 노드 모니터링 및 장애 발생 시 자동 장애 조치
- 클라이언트에 알림 및 새 마스터 정보 제공

```conf
# sentinel.conf 설정 예시
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

#### 2.4.3 Redis Cluster

- 데이터를 여러 노드에 분산하여 수평적 확장성 제공
- 노드 간 자동 데이터 분할 및 재배치
- 일부 노드 실패 시에도 클러스터 운영 가능

```conf
# cluster 설정 예시 (redis.conf)
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

## 3. Redis 도커 구성 분석

현재 프로젝트에서 사용 중인 Redis 도커 구성을 살펴보겠습니다:

### 3.1 Dockerfile 분석

```dockerfile
FROM redis:7.0

EXPOSE 6379

# 필요시 redis.conf 커스텀
# COPY redis.conf /usr/local/etc/redis/redis.conf
# CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]

CMD ["redis-server"]
```

구성 요소:

- **베이스 이미지**: Redis 7.0 공식 이미지 사용
- **포트 노출**: 기본 Redis 포트인 6379 노출
- **실행 명령**: 기본 설정으로 `redis-server` 실행
- **주석 처리된 옵션**: 커스텀 설정 파일 사용 방법 제시

### 3.2 docker-compose.yml 설정

```yaml
services:
  redis:
    build:
      context: ../..
      dockerfile: docker/redis/Dockerfile-redis
    container_name: redis_prod
    environment:
      TZ: Asia/Seoul
    ports:
      - "6379:6379"
```

주요 설정:

- **시간대 설정**: 한국 시간대(Asia/Seoul) 사용
- **포트 매핑**: 호스트의 6379 포트를 컨테이너의 6379 포트에 매핑
- **볼륨 마운트**: 별도의 볼륨 설정 없음 (기본 메모리 저장소로 사용)

## 4. Redis 최적화 및 운영 가이드

### 4.1 성능 최적화 전략

#### 4.1.1 메모리 관리

- **maxmemory 설정**: 사용 가능한 최대 메모리 제한
- **maxmemory-policy**: 메모리 한도 도달 시 제거 정책 설정
  - `noeviction`: 쓰기 작업 실패
  - `allkeys-lru`: 가장 최근에 사용되지 않은 키 제거
  - `volatile-lru`: 만료 시간이 설정된 키 중 LRU 방식으로 제거
  - `allkeys-random`: 무작위 키 제거
  - `volatile-random`: 만료 시간이 설정된 키 중 무작위 제거
  - `volatile-ttl`: 만료 시간이 가장 짧은 키 제거

```conf
# redis.conf 설정 예시
maxmemory 2gb
maxmemory-policy allkeys-lru
```

#### 4.1.2 키 설계 최적화

- **짧은 키 이름 사용**: 메모리 효율 향상
- **네임스페이스 패턴 사용**: `object-type:id:field` 형식 (예: `user:1000:email`)
- **압축 데이터 구조 사용**: Redis 5.0부터 지원하는 Stream과 같은 압축 구조 활용

#### 4.1.3 명령 실행 최적화

- **파이프라이닝**: 여러 명령을 한 번에 전송하여 네트워크 왕복 시간 절약
- **트랜잭션**: 여러 명령을 원자적으로 실행
- **Lua 스크립트**: 복잡한 작업을 서버 측에서 단일 호출로 실행

#### 4.1.4 연결 관리

- **연결 풀링**: 클라이언트 라이브러리에서 연결 풀 사용
- **UNLINK 명령**: 블로킹 없이 키 삭제 (Redis 4.0+)
- **Lazy Free**: 백그라운드에서 메모리 회수

### 4.2 캐싱 전략

#### 4.2.1 일반적인 캐싱 패턴

- **Cache-Aside (Lazy Loading)**

  - 데이터 요청 시 캐시 확인 후 없으면 DB에서 가져와 캐시에 저장
  - 필요한 데이터만 캐시, 초기 지연 발생 가능

  ```
  데이터 = 캐시에서 가져오기
  IF 데이터 == NULL THEN
      데이터 = DB에서 가져오기
      캐시에 데이터 저장
  END
  ```

- **Write-Through**

  - DB 쓰기 작업마다 캐시도 함께 업데이트
  - 캐시와 DB 일관성 유지, 쓰기 지연 발생

  ```
  데이터를 DB에 저장
  데이터를 캐시에 저장
  ```

- **Write-Behind (Write-Back)**
  - 캐시에 먼저 쓰고 나중에 비동기적으로 DB에 저장
  - 쓰기 성능 향상, 데이터 손실 위험
  ```
  데이터를 캐시에 저장
  작업을 큐에 추가
  백그라운드 프로세스:
      큐에서 작업 가져와 DB에 저장
  ```

#### 4.2.2 만료 정책 설정

- **TTL 기반 만료**: 키 별로 적절한 만료 시간 설정
- **LRU 관리**: `maxmemory-policy`를 사용한 자동 관리
- **주기적 갱신**: 백그라운드 작업으로 캐시 데이터 갱신

#### 4.2.3 캐시 무효화 전략

- **명시적 삭제**: 데이터 변경 시 관련 캐시 키 삭제
- **버전 관리**: 키에 버전 정보 포함하여 업데이트 시 버전 증가
- **패턴 삭제**: 관련된 여러 키를 한 번에 삭제 (`SCAN`과 `DEL` 조합)

### 4.3 모니터링 및 유지보수

#### 4.3.1 주요 모니터링 지표

- **메모리 사용량**: `used_memory`, `used_memory_rss`
- **히트율**: `keyspace_hits` / (`keyspace_hits` + `keyspace_misses`)
- **명령 처리량**: `total_commands_processed`
- **연결 수**: `connected_clients`
- **복제 상태**: `master_link_status`, `master_last_io_seconds_ago`

#### 4.3.2 모니터링 도구 및 명령

- **INFO 명령**: 서버 상태 정보 제공

  ```bash
  redis-cli info
  redis-cli info memory
  redis-cli info stats
  ```

- **MONITOR 명령**: 실시간 명령 모니터링 (주의: 성능 영향)

  ```bash
  redis-cli monitor
  ```

- **Prometheus + Grafana**: 메트릭 수집 및 시각화
- **Redis Exporter**: Prometheus용 Redis 메트릭 수집기

#### 4.3.3 관리 및 유지보수 작업

- **CONFIG 명령**: 런타임에 설정 변경

  ```bash
  redis-cli config set maxmemory 4gb
  ```

- **SAVE/BGSAVE**: 수동 RDB 스냅샷 생성

  ```bash
  redis-cli bgsave
  ```

- **BGREWRITEAOF**: AOF 파일 최적화

  ```bash
  redis-cli bgrewriteaof
  ```

- **FLUSHDB/FLUSHALL**: 데이터베이스 또는 전체 인스턴스 비우기 (주의 필요)
  ```bash
  redis-cli flushdb
  ```

## 5. Redis 활용 사례 및 응용

### 5.1 캐싱 활용

#### 5.1.1 데이터 캐싱

Spring Boot 애플리케이션에서 Redis를 사용한 데이터 캐싱 예시:

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private RedisTemplate<String, Product> redisTemplate;

    private static final String CACHE_KEY_PREFIX = "product:";
    private static final long CACHE_TTL = 3600; // 1시간

    public Product getProductById(Long id) {
        String cacheKey = CACHE_KEY_PREFIX + id;

        // 캐시에서 조회
        Product cachedProduct = redisTemplate.opsForValue().get(cacheKey);
        if (cachedProduct != null) {
            return cachedProduct;
        }

        // DB에서 조회
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));

        // 캐시에 저장
        redisTemplate.opsForValue().set(cacheKey, product, CACHE_TTL, TimeUnit.SECONDS);

        return product;
    }

    public void updateProduct(Product product) {
        // DB 업데이트
        productRepository.save(product);

        // 캐시 갱신
        String cacheKey = CACHE_KEY_PREFIX + product.getId();
        redisTemplate.opsForValue().set(cacheKey, product, CACHE_TTL, TimeUnit.SECONDS);
    }

    public void deleteProduct(Long id) {
        // DB에서 삭제
        productRepository.deleteById(id);

        // 캐시에서 삭제
        redisTemplate.delete(CACHE_KEY_PREFIX + id);
    }
}
```

#### 5.1.2 세션 관리

Spring Session과 Redis를 사용한 세션 관리:

```java
// Spring Boot 애플리케이션 설정
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800) // 30분
public class SessionConfig {

    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory("redis", 6379);
    }
}

// 세션 데이터 사용 예시
@RestController
public class UserController {

    @GetMapping("/api/user")
    public UserInfo getUserInfo(HttpSession session) {
        UserInfo userInfo = (UserInfo) session.getAttribute("userInfo");
        if (userInfo == null) {
            throw new UnauthorizedException("Not logged in");
        }
        return userInfo;
    }

    @PostMapping("/api/login")
    public void login(@RequestBody LoginRequest request, HttpSession session) {
        // 로그인 로직...
        UserInfo userInfo = authenticateUser(request);

        // 세션에 사용자 정보 저장
        session.setAttribute("userInfo", userInfo);
    }

    @PostMapping("/api/logout")
    public void logout(HttpSession session) {
        session.invalidate();
    }
}
```

### 5.2 메시지 브로커 및 발행/구독

#### 5.2.1 실시간 알림 시스템

Redis Pub/Sub을 활용한 실시간 알림 시스템:

```java
// 메시지 발행자 서비스
@Service
public class NotificationService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void sendNotification(String userId, String message) {
        NotificationMessage notification = new NotificationMessage(
            userId, message, System.currentTimeMillis()
        );

        String channel = "notifications:" + userId;
        redisTemplate.convertAndSend(channel,
            new ObjectMapper().writeValueAsString(notification));
    }
}

// 메시지 구독자 구성
@Configuration
public class RedisSubscriberConfig {

    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(
            RedisConnectionFactory connectionFactory,
            NotificationListener notificationListener) {

        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);

        // 모든 사용자의 알림 채널 패턴 구독
        container.addMessageListener(notificationListener,
            new PatternTopic("notifications:*"));

        return container;
    }
}

// 메시지 리스너
@Component
public class NotificationListener implements MessageListener {

    @Autowired
    private SimpMessagingTemplate websocketMessagingTemplate;

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String channel = new String(message.getChannel());
        String userId = channel.split(":")[1];

        // WebSocket을 통해 클라이언트에게 메시지 전달
        websocketMessagingTemplate.convertAndSend(
            "/topic/notifications/" + userId,
            new String(message.getBody()));
    }
}
```

#### 5.2.2 WebSocket을 사용한 채팅 애플리케이션

Redis와 WebSocket을 결합한 채팅 애플리케이션:

```java
// 채팅 메시지 발행 서비스
@Service
public class ChatService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void sendMessage(String roomId, ChatMessage message) {
        // 채팅 기록 저장 (Redis List 활용)
        String chatHistoryKey = "chat:history:" + roomId;
        redisTemplate.opsForList().rightPush(chatHistoryKey,
            new ObjectMapper().writeValueAsString(message));

        // 최대 100개 메시지만 유지
        redisTemplate.opsForList().trim(chatHistoryKey, -100, -1);

        // 채팅방 구독자에게 메시지 발행
        String channel = "chat:room:" + roomId;
        redisTemplate.convertAndSend(channel,
            new ObjectMapper().writeValueAsString(message));
    }

    public List<ChatMessage> getChatHistory(String roomId, int count) {
        String chatHistoryKey = "chat:history:" + roomId;
        List<String> messages = redisTemplate.opsForList()
            .range(chatHistoryKey, -count, -1);

        // JSON 문자열을 ChatMessage 객체로 변환
        return messages.stream()
            .map(msg -> {
                try {
                    return new ObjectMapper().readValue(msg, ChatMessage.class);
                } catch (Exception e) {
                    return null;
                }
            })
            .filter(Objects::nonNull)
            .collect(Collectors.toList());
    }
}

// WebSocket 컨트롤러
@Controller
public class ChatController {

    @Autowired
    private ChatService chatService;

    @MessageMapping("/chat.sendMessage/{roomId}")
    @SendTo("/topic/chat/{roomId}")
    public ChatMessage sendMessage(@DestinationVariable String roomId,
                                  ChatMessage message) {
        message.setTimestamp(System.currentTimeMillis());
        chatService.sendMessage(roomId, message);
        return message;
    }

    @GetMapping("/api/chat/{roomId}/history")
    public ResponseEntity<List<ChatMessage>> getChatHistory(
            @PathVariable String roomId,
            @RequestParam(defaultValue = "50") int count) {
        return ResponseEntity.ok(chatService.getChatHistory(roomId, count));
    }
}
```

### 5.3 분산 락 및 레이트 리미팅

#### 5.3.1 분산 락 구현

여러 서버에서 공유 리소스 접근을 조정하기 위한 분산 락:

```java
@Service
public class RedisDistributedLockService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    private static final String LOCK_PREFIX = "lock:";
    private static final long DEFAULT_EXPIRY = 30000; // 30초

    /**
     * 분산 락 획득 시도
     * @param resourceId 락을 획득할 리소스 ID
     * @param expiryMillis 락 만료 시간(밀리초)
     * @return 락 획득 성공 여부
     */
    public boolean acquireLock(String resourceId, long expiryMillis) {
        String lockKey = LOCK_PREFIX + resourceId;
        String lockValue = UUID.randomUUID().toString();

        // SET NX EX 명령어로 락 획득 시도
        Boolean acquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, expiryMillis, TimeUnit.MILLISECONDS);

        return acquired != null && acquired;
    }

    /**
     * 락 해제
     * @param resourceId 락을 해제할 리소스 ID
     * @return 락 해제 성공 여부
     */
    public boolean releaseLock(String resourceId) {
        String lockKey = LOCK_PREFIX + resourceId;

        // 락이 존재하는지 확인 후 삭제
        Boolean exists = redisTemplate.hasKey(lockKey);
        if (exists != null && exists) {
            redisTemplate.delete(lockKey);
            return true;
        }
        return false;
    }

    /**
     * 락을 획득하고 작업 실행 후 락 해제
     * @param resourceId 락을 획득할 리소스 ID
     * @param task 실행할 작업
     * @param maxRetries 최대 재시도 횟수
     * @return 작업 결과
     */
    public <T> T executeWithLock(String resourceId, Supplier<T> task,
                                int maxRetries) throws InterruptedException {
        int retries = 0;

        while (retries < maxRetries) {
            if (acquireLock(resourceId, DEFAULT_EXPIRY)) {
                try {
                    return task.get();
                } finally {
                    releaseLock(resourceId);
                }
            }

            // 백오프 시간 0.1초 ~ 1초
            long backoffMillis = Math.min(100 * (1L << retries), 1000);
            Thread.sleep(backoffMillis);
            retries++;
        }

        throw new RuntimeException("Failed to acquire lock after " + maxRetries + " retries");
    }
}
```

#### 5.3.2 레이트 리미팅

API 요청 제한을 위한 레이트 리미팅 구현:

```java
@Component
public class RedisRateLimiter {

    @Autowired
    private StringRedisTemplate redisTemplate;

    /**
     * 단순 카운터 기반 레이트 리미터
     * @param key 제한할 키 (예: 사용자 ID 또는 IP)
     * @param limit 허용 요청 수
     * @param windowSeconds 시간 창(초)
     * @return 요청 허용 여부
     */
    public boolean isAllowed(String key, int limit, int windowSeconds) {
        String redisKey = "ratelimit:" + key + ":" + (System.currentTimeMillis() / (windowSeconds * 1000));

        Long count = redisTemplate.opsForValue().increment(redisKey, 1);
        if (count == 1) {
            redisTemplate.expire(redisKey, windowSeconds, TimeUnit.SECONDS);
        }

        return count <= limit;
    }

    /**
     * 슬라이딩 윈도우 레이트 리미터
     * @param key 제한할 키 (예: 사용자 ID 또는 IP)
     * @param limit 허용 요청 수
     * @param windowSeconds 시간 창(초)
     * @return 요청 허용 여부
     */
    public boolean isAllowedSlidingWindow(String key, int limit, int windowSeconds) {
        String redisKey = "ratelimit:sliding:" + key;
        long currentTime = System.currentTimeMillis();
        long windowStart = currentTime - (windowSeconds * 1000);

        // 현재 시간을 점수로 하여 정렬된 집합에 추가
        redisTemplate.opsForZSet().add(redisKey, UUID.randomUUID().toString(), currentTime);

        // 윈도우 밖의 요청 제거
        redisTemplate.opsForZSet().removeRangeByScore(redisKey, 0, windowStart);

        // 현재 윈도우 내의 요청 수 계산
        Long count = redisTemplate.opsForZSet().zCard(redisKey);

        // 키 만료 시간 설정 (윈도우 크기 + 1초)
        redisTemplate.expire(redisKey, windowSeconds + 1, TimeUnit.SECONDS);

        return count <= limit;
    }
}

// 레이트 리미팅을 적용하는 Spring 인터셉터
@Component
public class RateLimitInterceptor implements HandlerInterceptor {

    @Autowired
    private RedisRateLimiter rateLimiter;

    private static final int DEFAULT_LIMIT = 100;  // 기본 요청 제한
    private static final int DEFAULT_WINDOW = 60;  // 기본 윈도우 크기(초)

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler) throws Exception {

        // 사용자 식별자 (IP 또는 인증된 사용자 ID)
        String identifier = getIdentifier(request);

        // API 엔드포인트별 서로 다른 제한 적용
        String endpoint = request.getRequestURI();
        if (endpoint.startsWith("/api/public/")) {
            // 공개 API: 더 관대한 제한
            if (!rateLimiter.isAllowedSlidingWindow(identifier + ":public", 200, 60)) {
                response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
                response.getWriter().write("Rate limit exceeded. Try again later.");
                return false;
            }
        } else if (endpoint.startsWith("/api/sensitive/")) {
            // 민감한 API: 더 엄격한 제한
            if (!rateLimiter.isAllowedSlidingWindow(identifier + ":sensitive", 10, 60)) {
                response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
                response.getWriter().write("Rate limit exceeded for sensitive operation.");
                return false;
            }
        } else {
            // 일반 API: 기본 제한
            if (!rateLimiter.isAllowedSlidingWindow(identifier, DEFAULT_LIMIT, DEFAULT_WINDOW)) {
                response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
                response.getWriter().write("Rate limit exceeded. Try again later.");
                return false;
            }
        }

        return true;
    }

    private String getIdentifier(HttpServletRequest request) {
        // 인증된 사용자가 있으면 사용자 ID 사용
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth != null && auth.isAuthenticated() && !(auth instanceof AnonymousAuthenticationToken)) {
            return "user:" + auth.getName();
        }

        // 그렇지 않으면 IP 주소 사용
        return "ip:" + request.getRemoteAddr();
    }
}
```

### 5.4 Redis 성능 캐싱과 애플리케이션 통합

#### 5.4.1 Nginx 캐시 퍼지와 Redis 통합

Nginx에서 캐시 무효화를 처리하기 위한 Redis 기반 시스템:

```java
@Service
public class CacheInvalidationService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    private final RestTemplate restTemplate = new RestTemplate();
    private static final String NGINX_PURGE_ENDPOINT = "http://nginx:80/purge_products/";

    /**
     * 제품 ID에 대한 캐시 무효화
     * @param productId 캐시를 무효화할 제품 ID
     */
    public void invalidateProductCache(Long productId) {
        // Redis에 무효화 이벤트 발행
        String channel = "cache:invalidation:products";
        String message = String.valueOf(productId);
        redisTemplate.convertAndSend(channel, message);

        // Nginx 캐시 퍼지 요청
        try {
            String purgeUrl = NGINX_PURGE_ENDPOINT + "/api/products/query/" + productId;
            restTemplate.exchange(purgeUrl, HttpMethod.GET, null, String.class);

            // 상세 페이지 퍼지
            purgeUrl = NGINX_PURGE_ENDPOINT + "/api/products/query/" + productId + "/detail";
            restTemplate.exchange(purgeUrl, HttpMethod.GET, null, String.class);

            // 이미지 퍼지
            purgeUrl = NGINX_PURGE_ENDPOINT + "/api/products/query/" + productId + "/images";
            restTemplate.exchange(purgeUrl, HttpMethod.GET, null, String.class);
        } catch (Exception e) {
            // 로깅 또는 재시도 로직
        }
    }

    /**
     * 카테고리에 대한 캐시 무효화
     * @param categoryId 캐시를 무효화할 카테고리 ID
     */
    public void invalidateCategoryCache(Long categoryId) {
        // Redis에 무효화 이벤트 발행
        String channel = "cache:invalidation:categories";
        String message = String.valueOf(categoryId);
        redisTemplate.convertAndSend(channel, message);

        // Nginx 캐시 퍼지 요청
        try {
            String purgeUrl = NGINX_PURGE_ENDPOINT + "/api/products/query?category=" + categoryId;
            restTemplate.exchange(purgeUrl, HttpMethod.GET, null, String.class);
        } catch (Exception e) {
            // 로깅 또는 재시도 로직
        }
    }
}

// 캐시 무효화 이벤트 리스너
@Component
public class CacheInvalidationListener {

    private static final Logger logger = LoggerFactory.getLogger(CacheInvalidationListener.class);

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @PostConstruct
    public void init() {
        // 캐시 무효화 채널 구독
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisTemplate.getConnectionFactory());

        // 제품 캐시 무효화 리스너
        container.addMessageListener((message, pattern) -> {
            String productId = new String(message.getBody());
            logger.info("Received product cache invalidation for product ID: {}", productId);
            // 애플리케이션 내부 캐시 무효화 처리
        }, new ChannelTopic("cache:invalidation:products"));

        // 카테고리 캐시 무효화 리스너
        container.addMessageListener((message, pattern) -> {
            String categoryId = new String(message.getBody());
            logger.info("Received category cache invalidation for category ID: {}", categoryId);
            // 애플리케이션 내부 캐시 무효화 처리
        }, new ChannelTopic("cache:invalidation:categories"));

        container.start();
    }
}
```

#### 5.4.2 로그인 시도 제한 및 보안

Redis를 사용한 로그인 시도 제한 및 보안 기능:

```java
@Service
public class AuthenticationSecurityService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    private static final int MAX_LOGIN_ATTEMPTS = 5;
    private static final int ACCOUNT_LOCK_TIME_MINUTES = 30;

    /**
     * 로그인 실패 카운트 증가 및 계정 잠금 상태 확인
     * @param username 사용자명
     * @return 계정 상태 (정상, 잠금됨, 경고)
     */
    public AccountStatus checkLoginAttempts(String username) {
        String attemptKey = "auth:loginfail:" + username;
        String lockKey = "auth:accountlock:" + username;

        // 계정 잠금 상태 확인
        Boolean isLocked = redisTemplate.hasKey(lockKey);
        if (isLocked != null && isLocked) {
            // 계정이 잠긴 경우 남은 시간 확인
            Long ttl = redisTemplate.getExpire(lockKey, TimeUnit.MINUTES);
            if (ttl != null && ttl > 0) {
                return new AccountStatus(AccountStatusType.LOCKED, ttl);
            }
        }

        // 로그인 실패 횟수 확인
        String countStr = redisTemplate.opsForValue().get(attemptKey);
        int failCount = (countStr != null) ? Integer.parseInt(countStr) : 0;

        if (failCount >= MAX_LOGIN_ATTEMPTS - 1) {
            return new AccountStatus(AccountStatusType.WARNING,
                MAX_LOGIN_ATTEMPTS - failCount);
        }

        return new AccountStatus(AccountStatusType.NORMAL, null);
    }

    /**
     * 로그인 실패 처리
     * @param username 사용자명
     * @return 계정이 잠겼는지 여부
     */
    public boolean handleFailedLogin(String username) {
        String attemptKey = "auth:loginfail:" + username;
        String lockKey = "auth:accountlock:" + username;

        // 실패 카운트 증가
        Long count = redisTemplate.opsForValue().increment(attemptKey, 1);
        if (count == 1) {
            // 첫 실패 시 24시간 후 초기화되도록 설정
            redisTemplate.expire(attemptKey, 24, TimeUnit.HOURS);
        }

        // 최대 시도 횟수 초과 시 계정 잠금
        if (count >= MAX_LOGIN_ATTEMPTS) {
            redisTemplate.opsForValue().set(lockKey, "locked",
                ACCOUNT_LOCK_TIME_MINUTES, TimeUnit.MINUTES);
            return true;
        }

        return false;
    }

    /**
     * 로그인 성공 처리
     * @param username 사용자명
     */
    public void handleSuccessfulLogin(String username) {
        String attemptKey = "auth:loginfail:" + username;

        // 로그인 실패 카운트 초기화
        redisTemplate.delete(attemptKey);
    }

    /**
     * 계정 잠금 해제 (관리자 기능)
     * @param username 사용자명
     */
    public void unlockAccount(String username) {
        String lockKey = "auth:accountlock:" + username;
        String attemptKey = "auth:loginfail:" + username;

        // 잠금 및 실패 카운트 제거
        redisTemplate.delete(lockKey);
        redisTemplate.delete(attemptKey);
    }

    // 계정 상태 반환 클래스
    public static class AccountStatus {
        private AccountStatusType type;
        private Long value;

        public AccountStatus(AccountStatusType type, Long value) {
            this.type = type;
            this.value = value;
        }

        // Getters
    }

    public enum AccountStatusType {
        NORMAL, WARNING, LOCKED
    }
}
```

## 6. 문제 해결 및 디버깅 가이드

### 6.1 일반적인 문제와 해결책

#### 6.1.1 메모리 관련 문제

- **메모리 부족**: `maxmemory` 설정 조정 또는 메모리 사용량이 많은 키 식별하여 정리
- **단편화**: 재시작으로 메모리 압축 또는 `MEMORY PURGE` 명령 사용 (Redis 4.0+)
- **키 크기 최적화**: 대용량 키 분할, 압축 데이터 구조 사용

#### 6.1.2 성능 문제

- **느린 명령어**: `SLOWLOG` 명령으로 느린 쿼리 식별 및 최적화
- **복잡한 Lua 스크립트**: 스크립트 최적화 또는 더 작은 단위로 분할
- **네트워크 지연**: Redis 서버와 클라이언트 간 네트워크 최적화

#### 6.1.3 연결 관련 문제

- **최대 연결 수 초과**: `maxclients` 설정 증가 또는 연결 풀링 사용
- **연결 누수**: 클라이언트에서 연결 자원 관리 개선
- **연결 시간 초과**: 타임아웃 설정 조정 및 네트워크 상태 확인

### 6.2 디버깅 도구 및 명령

#### 6.2.1 INFO 명령

서버의 다양한 정보를 얻을 수 있는 명령:

```bash
# 기본 정보
redis-cli info

# 메모리 정보
redis-cli info memory

# 클라이언트 연결 정보
redis-cli info clients

# 통계 정보
redis-cli info stats

# 복제 정보
redis-cli info replication
```

#### 6.2.2 MONITOR 명령

실시간으로 Redis 서버에 수신된 명령 모니터링:

```bash
redis-cli monitor
```

⚠️ 주의: 프로덕션 환경에서는 짧은 시간만 사용할 것. 성능에 영향을 미칠 수 있음.

#### 6.2.3 SLOWLOG 명령

실행 시간이 긴 명령들의 로그 확인:

```bash
# 느린 쿼리 로그 설정
redis-cli config set slowlog-log-slower-than 10000 # 10ms 이상인 쿼리 기록
redis-cli config set slowlog-max-len 128 # 최대 128개까지 기록

# 느린 쿼리 로그 조회
redis-cli slowlog get 10 # 최근 10개의 느린 쿼리
redis-cli slowlog len # 현재 저장된 느린 쿼리 수
redis-cli slowlog reset # 느린 쿼리 로그 초기화
```

#### 6.2.4 MEMORY 명령

메모리 사용량과 관련된 문제 진단:

```bash
# 키의 메모리 사용량 확인
redis-cli memory usage my_key

# 가장 많은 메모리를 사용하는 키 샘플링
redis-cli --bigkeys

# 메모리 사용량 통계
redis-cli memory stats

# 메모리 닥터 (메모리 사용 관련 조언)
redis-cli memory doctor
```

### 6.3 Docker 환경에서의 디버깅

#### 6.3.1 로그 확인

```bash
# 컨테이너 로그 확인
docker logs redis_prod

# 실시간 로그 추적
docker logs -f redis_prod
```

#### 6.3.2 컨테이너 내부 접속

```bash
# Redis 컨테이너에 접속
docker exec -it redis_prod bash

# Redis CLI 실행
docker exec -it redis_prod redis-cli
```

#### 6.3.3 Redis 벤치마크 도구 사용

```bash
# Redis 벤치마크 실행 (100,000개 요청, 50개 병렬 클라이언트)
docker exec -it redis_prod redis-benchmark -n 100000 -c 50

# 특정 명령에 대한 벤치마크
docker exec -it redis_prod redis-benchmark -t get,set -n 100000 -c 50

# 다양한 크기의 값에 대한 벤치마크
docker exec -it redis_prod redis-benchmark -t set -n 100000 -r 100000 -d 2048
```

## 7. 참고 자료 및 추가 학습 리소스

### 7.1 공식 문서

- [Redis 공식 문서](https://redis.io/documentation)
- [Redis 명령어 레퍼런스](https://redis.io/commands)
- [Redis Docker 이미지](https://hub.docker.com/_/redis)
