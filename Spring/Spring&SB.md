# Spring Framework와 Spring Boot 비교 문서

## 1. Spring Framework(레거시)의 개념과 특징

### 1.1 개념

Spring Framework는 엔터프라이즈 애플리케이션 개발을 위한 자바 기반의 오픈소스 프레임워크로, 2003년 Rod Johnson에 의해 처음 출시되었습니다. Spring은 Java EE(Enterprise Edition)의 복잡성을 해결하고 POJO(Plain Old Java Object) 기반의 프로그래밍 모델을 통해 엔터프라이즈급 애플리케이션을 더 쉽게 개발할 수 있도록 설계되었습니다.

### 1.2 핵심 기능 및 특징

#### 1.2.1 IoC(Inversion of Control, 제어의 역전)

- 객체의 생성과 생명주기, 의존성 관리를 개발자가 아닌 프레임워크가 담당
- Spring Container가 Bean 객체들을 생성하고 관리하며, 필요한 곳에 주입
- ApplicationContext와 BeanFactory가 주요 IoC 컨테이너 인터페이스

```java
// IoC 컨테이너 초기화 예시
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService userService = context.getBean("userService", UserService.class);
```

#### 1.2.2 DI(Dependency Injection, 의존성 주입)

Spring에서 지원하는 주요 DI 방식:

1. **생성자 주입**

```java
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    @Autowired // Spring 4.3부터는 단일 생성자의 경우 @Autowired 생략 가능
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

2. **setter 주입**

```java
@Service
public class UserServiceImpl implements UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

3. **필드 주입**

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository userRepository;
}
```

#### 1.2.3 AOP(Aspect-Oriented Programming)

- 비즈니스 로직에서 부가 기능(로깅, 트랜잭션, 보안 등)을 분리하여 모듈화
- 주요 용어 상세 설명:
  - **Aspect**: 여러 클래스에 걸쳐 있는 관심사(concern)를 모듈화한 것 (@Aspect)
  - **Advice**: 특정 조인포인트에서 Aspect가 취하는 행동 (메소드)
    - @Before: 메소드 실행 전
    - @After: 메소드 실행 후 (성공/예외 상관없이)
    - @AfterReturning: 메소드가 정상적으로 반환된 후
    - @AfterThrowing: 메소드가 예외를 던진 후
    - @Around: 메소드 실행 전후를 감싸는 방식
  - **JoinPoint**: 프로그램 실행 중 Aspect가 적용될 수 있는 지점
  - **Pointcut**: 조인포인트의 부분 집합으로 Advice가 실행되는 시점 지정
  - **Weaving**: Aspect를 적용하는 과정

```java
// AOP 상세 예시
@Aspect
@Component
public class PerformanceAspect {
    private static final Logger logger = LoggerFactory.getLogger(PerformanceAspect.class);

    @Around("execution(* com.example.service.*.*(..))")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        try {
            return joinPoint.proceed();
        } finally {
            long executionTime = System.currentTimeMillis() - start;
            logger.info("{} executed in {} ms",
                     joinPoint.getSignature().toShortString(),
                     executionTime);
        }
    }
}
```

#### 1.2.4 모듈 구성

Spring Framework의 주요 모듈:

- **Spring Core**: IoC와 DI 기능 제공
- **Spring AOP**: 관점 지향 프로그래밍 지원
- **Spring JDBC**: JDBC 추상화 계층 제공
- **Spring ORM**: Hibernate, JPA와 같은 ORM 프레임워크 통합
- **Spring Web**: 웹 애플리케이션 개발을 위한 기본 기능 제공
- **Spring MVC**: 웹 MVC 패턴 구현을 위한 프레임워크
- **Spring Test**: Spring 애플리케이션의 단위 및 통합 테스트 지원

### 1.3 설정 방식

#### 1.3.1 XML 기반 설정

```xml
<!-- applicationContext.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userRepository" class="com.example.repository.UserRepositoryImpl" />

    <bean id="userService" class="com.example.service.UserServiceImpl">
        <constructor-arg ref="userRepository" />
    </bean>
</beans>
```

#### 1.3.2 Java Configuration

```java
@Configuration
public class AppConfig {

    @Bean
    public UserRepository userRepository() {
        return new UserRepositoryImpl();
    }

    @Bean
    public UserService userService() {
        return new UserServiceImpl(userRepository());
    }
}
```

#### 1.3.3 애노테이션 기반 설정

```java
// 컴포넌트 스캔 설정
@Configuration
@ComponentScan("com.example")
public class AppConfig { }

// 컴포넌트 클래스
@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 1.4 Spring MVC 아키텍처

Spring MVC는 Model-View-Controller 패턴을 구현한 웹 프레임워크로, 다음과 같은 컴포넌트로 구성됩니다:

- **DispatcherServlet**: 프론트 컨트롤러로 모든 요청을 받아 적절한 컨트롤러에 위임
- **HandlerMapping**: 요청 URL에 맞는 핸들러(컨트롤러) 매핑
- **Controller**: 비즈니스 로직 처리 및 모델 생성
- **ViewResolver**: 컨트롤러가 반환한 뷰 이름을 실제 뷰 객체로 변환
- **View**: 모델 데이터를 사용하여 결과 표시

```java
// Spring MVC 컨트롤러 예시
@Controller
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "user/detail"; // ViewResolver가 처리할 뷰 이름
    }
}
```

### 1.5 Spring 레거시 프로젝트 구조 상세

```
src/main/java
 └─com.example
   ├─controller    // MVC 패턴의 컨트롤러
   │ └─UserController.java
   ├─service       // 비즈니스 로직 계층
   │ ├─UserService.java (인터페이스)
   │ └─UserServiceImpl.java
   ├─repository    // 데이터 접근 계층
   │ ├─UserRepository.java (인터페이스)
   │ └─UserRepositoryImpl.java
   ├─domain        // 도메인 모델(엔티티)
   │ └─User.java
   ├─dto           // 데이터 전송 객체
   │ └─UserDTO.java
   ├─exception     // 사용자 정의 예외 클래스
   │ └─UserNotFoundException.java
   ├─aop           // AOP 관련 클래스
   │ └─LoggingAspect.java
   └─config        // Java 기반 설정 클래스
     ├─WebMvcConfig.java
     └─RootConfig.java

src/main/resources
 ├─applicationContext.xml  // 루트 애플리케이션 컨텍스트 설정
 ├─servlet-context.xml     // 서블릿 컨텍스트 설정
 ├─mybatis-config.xml      // MyBatis 설정 (사용시)
 ├─mappers/               // SQL 매퍼 (사용시)
 │ └─UserMapper.xml
 ├─logback.xml            // 로깅 설정
 └─db.properties          // 데이터베이스 연결 정보

src/main/webapp
 ├─WEB-INF
 │ ├─views/              // JSP 뷰 템플릿
 │ │ └─user/
 │ │   ├─list.jsp
 │ │   └─detail.jsp
 │ └─web.xml             // 웹 애플리케이션 배포 서술자
 ├─resources/            // 정적 리소스
 │ ├─css/
 │ ├─js/
 │ └─images/
 └─index.jsp             // 웰컴 페이지

src/test/java            // 테스트 코드
 └─com.example
   ├─controller
   ├─service
   └─repository

pom.xml                  // Maven 프로젝트 설정 파일
```

## 2. Spring Boot의 개념과 특징

### 2.1 개념

Spring Boot는 Spring Framework 기반 애플리케이션을 빠르게 개발하고 실행할 수 있도록 하는 프레임워크입니다. Spring의 복잡한 설정을 자동화하고, "convention over configuration(설정보다 관례)" 원칙을 따라 개발자가 비즈니스 로직에 집중할 수 있게 합니다.

### 2.2 핵심 기능 및 특징

#### 2.2.1 자동 설정(Auto Configuration)

- classpath에 있는 jar 의존성, 환경 변수, 기타 설정에 기반하여 필요한 빈을 자동으로 등록
- `@EnableAutoConfiguration` 또는 `@SpringBootApplication` 애노테이션으로 활성화

```java
@SpringBootApplication // @Configuration, @EnableAutoConfiguration, @ComponentScan을 포함
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### 2.2.2 스타터 의존성(Spring Boot Starters)

- 특정 기능에 필요한 의존성 그룹을 편리하게 추가할 수 있는 패키지
- 대표적인 스타터:
  - `spring-boot-starter-web`: 웹 애플리케이션 개발 (Spring MVC, Tomcat 등)
  - `spring-boot-starter-data-jpa`: JPA 데이터 접근 계층
  - `spring-boot-starter-security`: Spring Security 보안 기능
  - `spring-boot-starter-test`: 테스트 프레임워크 (JUnit, Mockito 등)

```xml
<!-- Maven pom.xml 예시 -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```

#### 2.2.3 내장 서버(Embedded Server)

- 별도의 웹 서버(Tomcat, Jetty, Undertow) 설치 없이 애플리케이션 실행 가능
- 기본적으로 Tomcat이 내장되어 있으며, 필요시 변경 가능

```java
// 내장 서버 커스터마이징 예시
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.setPort(9000);
    factory.addConnectorCustomizers(connector ->
        connector.setProperty("relaxedQueryChars", "|{}[]"));
    return factory;
}
```

#### 2.2.4 프로덕션 준비 기능(Production-Ready Features)

- **Spring Boot Actuator**: 애플리케이션 모니터링 및 관리를 위한 엔드포인트 제공
  - `/health`: 애플리케이션 상태 확인
  - `/metrics`: 메트릭 정보 제공
  - `/info`: 애플리케이션 정보 제공
  - `/env`: 환경 변수 정보 제공
- **Spring Boot DevTools**: 개발 생산성 향상 (자동 재시작, 브라우저 자동 새로고침 등)

```properties
# application.properties에서 Actuator 엔드포인트 설정
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```

#### 2.2.5 외부 설정(Externalized Configuration)

- 동일한 애플리케이션 코드로 다양한 환경에서 작동할 수 있도록 설정 외부화
- 설정 소스 우선순위:
  1. 명령줄 인수
  2. JVM 시스템 속성
  3. OS 환경 변수
  4. 프로필별 application-{profile}.properties 또는 YAML 파일
  5. 기본 application.properties 또는 YAML 파일

```properties
# application.properties 예시
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update

# 로깅 설정
logging.level.org.springframework.web=INFO
logging.level.com.example=DEBUG

# 서버 포트 설정
server.port=8080
```

```yaml
# application.yml 예시
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
  jpa:
    hibernate:
      ddl-auto: update

logging:
  level:
    org.springframework.web: INFO
    com.example: DEBUG

server:
  port: 8080
```

### 2.3 Spring Boot 프로젝트 구조 상세

```
src/main/java
 └─com.example.demo
   ├─DemoApplication.java      // 메인 애플리케이션 클래스
   ├─controller                // REST 컨트롤러
   │ └─UserController.java
   ├─service                   // 비즈니스 로직 계층
   │ ├─UserService.java
   │ └─UserServiceImpl.java
   ├─repository                // 데이터 접근 계층
   │ └─UserRepository.java     // Spring Data JPA 인터페이스
   ├─entity                    // JPA 엔티티
   │ └─User.java
   ├─dto                       // 데이터 전송 객체
   │ └─UserDTO.java
   ├─exception                 // 사용자 정의 예외 및 글로벌 예외 처리
   │ ├─CustomException.java
   │ └─GlobalExceptionHandler.java
   ├─config                    // 추가 설정 클래스
   │ ├─WebConfig.java
   │ └─SecurityConfig.java (필요시)
   └─util                      // 유틸리티 클래스
     └─DateUtils.java

src/main/resources
 ├─application.yml            // 기본 설정 파일
 ├─application-dev.yml        // 개발 환경 설정
 ├─application-prod.yml       // 운영 환경 설정
 ├─messages/                  // 다국어 메시지
 │ └─messages.properties
 └─static/                    // 정적 리소스
   ├─css/
   ├─js/
   └─images/

src/test/java                 // 테스트 코드
 └─com.example.demo
   ├─controller
   │ └─UserControllerTest.java
   ├─service
   │ └─UserServiceTest.java
   └─repository
     └─UserRepositoryTest.java

src/test/resources            // 테스트용 설정 파일
 └─application-test.yml

pom.xml                       // Maven 프로젝트 설정
```

## 3. Spring 레거시와 Spring Boot의 심층 비교

### 3.1 설정(Configuration) 비교

#### 3.1.1 웹 애플리케이션 초기화 구성

**Spring 레거시**:

```xml
<!-- web.xml -->
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/servlet-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/root-context.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

**Spring Boot**:

```java
// 자동 설정으로 대체됨 (web.xml 필요 없음)
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

#### 3.1.2 데이터베이스 연결 설정

**Spring 레거시**:

```xml
<!-- applicationContext.xml -->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driver}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource" />
</bean>
```

**Spring Boot**:

```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

#### 3.1.3 트랜잭션 관리

**Spring 레거시**:

```xml
<!-- applicationContext.xml -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>

<tx:annotation-driven transaction-manager="transactionManager" />
```

**Spring Boot**:

```java
// 자동 설정으로 대체됨 (@EnableTransactionManagement 자동 적용)
@Service
public class UserServiceImpl implements UserService {
    @Transactional
    public void updateUser(User user) {
        // 트랜잭션 관리 자동 적용
    }
}
```

### 3.2 개발 경험 비교

#### 3.2.1 프로젝트 초기화

**Spring 레거시**:

- Maven/Gradle 프로젝트 생성
- 필요한 의존성 수동 추가
- 필수 설정 파일 생성 (web.xml, applicationContext.xml 등)
- Spring MVC 설정 구성
- 빌드 및 배포 설정

**Spring Boot**:

- Spring Initializr(https://start.spring.io/) 사용
- 필요한 스타터 의존성 선택
- 자동 생성된 프로젝트로 바로 시작 가능

#### 3.2.2 개발 생산성

**Spring 레거시**:

- 수동 설정에 많은 시간 소요
- 의존성 충돌 해결 필요
- 별도의 WAS 설치 및 설정
- 프로젝트 실행 및 배포 과정 복잡

**Spring Boot**:

- 자동 설정으로 개발 시작 시간 단축
- 스타터로 의존성 충돌 최소화
- 내장 서버로 즉시 실행 가능
- Spring Boot DevTools로 개발 생산성 향상
  - 자동 재시작 (코드 변경 감지)
  - LiveReload 지원 (브라우저 자동 새로고침)
  - 향상된 개발자 경험을 위한 기능

### 3.3 애플리케이션 아키텍처 비교

#### 3.3.1 계층 구조

**Spring 레거시**:

- 전통적인 계층형 아키텍처 (MVC)
- XML 기반 설정으로 인한 복잡성
- 컴포넌트 간 높은 결합도 가능성
- 대규모 모놀리식 애플리케이션에 적합

**Spring Boot**:

- 마이크로서비스 지향 설계 지원
- Java 설정 및 애노테이션 기반
- 모듈화 및 느슨한 결합 촉진
- 클라우드 네이티브 애플리케이션 개발에 유리

#### 3.3.2 REST API 개발

**Spring 레거시**:

```java
@Controller
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    @ResponseBody
    public UserDTO getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return convertToDTO(user);
    }

    private UserDTO convertToDTO(User user) {
        // 변환 로직
    }
}
```

**Spring Boot**:

```java
@RestController // @Controller + @ResponseBody
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) { // 생성자 주입 (자동 와이어링)
        this.userService = userService;
    }

    @GetMapping("/{id}") // 더 간결한 매핑 애노테이션
    public UserDTO getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

### 3.4 배포 및 운영 비교

#### 3.4.1 배포 방식

**Spring 레거시**:

- WAR 파일로 패키징
- 외부 WAS(Tomcat, JBoss, WebLogic 등)에 배포
- 서버 설정 및 관리 필요

**Spring Boot**:

- 실행 가능한 JAR 파일로 패키징 (내장 서버 포함)
- `java -jar application.jar` 명령으로 간단히 실행
- 컨테이너화(Docker) 용이

#### 3.4.2 모니터링 및 관리

**Spring 레거시**:

- 별도의 모니터링 도구 통합 필요
- 커스텀 상태 체크 API 구현
- 로깅 및 메트릭 수집 설정 필요

**Spring Boot**:

- Spring Boot Actuator를 통한 내장 모니터링 기능
- 다양한 엔드포인트로 애플리케이션 상태 확인 가능
  - `/actuator/health`: 상태 확인
  - `/actuator/metrics`: 메트릭 정보
  - `/actuator/loggers`: 로깅 레벨 조회/변경
- 마이크로미터(Micrometer) 통합으로 다양한 모니터링 시스템 지원
  - Prometheus, Grafana, ELK Stack 등

### 3.5 성능 및 확장성 비교

#### 3.5.1 시작 시간

**Spring 레거시**:

- 대규모 XML 파싱으로 인한 느린 시작 시간
- 컴포넌트 스캔 및 빈 초기화에 시간 소요
- 수동 최적화 필요

**Spring Boot**:

- 조건부 자동 설정으로 빠른 시작 시간
- Lazy Initialization 옵션 지원
- GraalVM 네이티브 이미지 지원으로 초고속 시작 가능 (Spring Native)

#### 3.5.2 리소스 사용

**Spring 레거시**:

- 외부 WAS로 인한 추가 리소스 사용
- 불필요한 컴포넌트 로드 가능성
- 수동 최적화 필요

**Spring Boot**:

- 필요한 컴포넌트만 로드하는 효율적인 설계
- 가볍고 최적화된 내장 서버
- 클라우드 환경에 최적화된 리소스 사용

## 4. 전환 및 마이그레이션 전략

### 4.1 Spring 레거시에서 Spring Boot로 전환하는 단계적 접근법

1. **의존성 관리 변경**

   - Maven/Gradle 빌드 파일 업데이트
   - Spring Boot 스타터 의존성 도입
   - 버전 관리를 Spring Boot 의존성 관리로 이전

2. **설정 파일 변환**

   - XML 설정을 Java 기반 설정으로 변환
   - 외부화된 속성을 application.properties/yml로 이동
   - 환경별 프로필 설정 도입

3. **애플리케이션 초기화 변경**

   - web.xml 제거 및 `@SpringBootApplication` 도입
   - 메인 클래스 생성 및 `SpringApplication.run()` 메소드 추가
   - 내장 서버 설정

4. **컴포넌트 스캔 및 자동 설정 조정**

   - 기존 컴포넌트 스캔 설정 확인 및 조정
   - 불필요한 자동 설정 비활성화 (`@EnableAutoConfiguration(exclude = {})`)
   - 커스텀 자동 설정 추가 (필요시)

5. **배포 방식 변경**
   - WAR에서 실행 가능한 JAR로 전환
   - 배포 스크립트 및 프로세스 업데이트
   - 컨테이너화 고려 (Docker)

### 4.2 점진적 마이그레이션 전략

1. **하이브리드 접근법**

   - Spring Boot와 레거시 Spring을 함께 실행
   - Spring Boot를 기존 WAR 내부에서 실행하는 방법으로 시작

   ```java
   @SpringBootApplication
   public class Application extends SpringBootServletInitializer {
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
           return application.sources(Application.class);
       }

       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   }
   ```

2. **기능별 마이그레이션**

   - 새로운 기능은 Spring Boot로 개발
   - 기존 기능은 점진적으로 마이그레이션
   - API 게이트웨이를 통한 라우팅 고려

3. **마이크로서비스 전환 고려**
   - 대규모 모놀리식 애플리케이션을 마이크로서비스로 분해
   - 각 마이크로서비스를 Spring Boot로 구현
   - 서비스 간 통신을 위한 전략 수립 (REST, 메시징 등)

## 5. 심화 개념 비교

### 5.1 반응형 프로그래밍 지원

#### 5.1.1 Spring 레거시

- Spring Framework 5.0부터 반응형 프로그래밍 지원 (WebFlux)
- 별도 설정 및 의존성 추가 필요
- 기존 서블릿 기반 애플리케이션과 함께 사용 어려움

#### 5.1.2 Spring Boot

- Spring Boot 2.0+ 에서 WebFlux 스타터 제공
- Reactive 스택 자동 설정
- Netty 서버 내장 (비차단 I/O 모델)
- 간단한 설정으로 반응형 애플리케이션 개발 가능

```java
// Spring Boot WebFlux 예시
@RestController
public class UserController {
    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping("/users")
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }

    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable String id) {
        return userRepository.findById(id);
    }
}
```

### 5.2 클라우드 네이티브 지원

#### 5.2.1 Spring 레거시

- 클라우드 환경에서 실행 가능하지만 추가 설정 필요
- 서드파티 라이브러리 통합 복잡
- 컨테이너화 및 오케스트레이션에 부가적인 작업 필요

#### 5.2.2 Spring Boot

- Spring Cloud 프로젝트와 긴밀한 통합
- 클라우드 설정 자동화 (서비스 디스커버리, 설정 관리, 서킷 브레이커 등)
- Kubernetes와 같은 오케스트레이션 플랫폼에 최적화
- 12-Factor 애플리케이션 원칙 준수 용이

```yaml
# Spring Boot Kubernetes 프로퍼티 예시
spring:
  application:
    name: user-service
  cloud:
    kubernetes:
      config:
        enabled: true
      reload:
        enabled: true
      secrets:
        enabled: true
```

### 5.3 보안 기능 비교

#### 5.3.1 Spring 레거시

```xml
<!-- Spring Security 설정 (XML) -->
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="...">

    <http auto-config="true">
        <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')" />
        <form-login login-page="/login" default-target-url="/home" />
        <logout logout-success-url="/login?logout" />
    </http>

    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="password" authorities="ROLE_ADMIN" />
            </user-service>
        </authentication-provider>
    </authentication-manager>
</beans:beans>
```

#### 5.3.2 Spring Boot

```java
// Spring Boot Security 설정
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .formLogin()
                .loginPage("/login")
                .defaultSuccessUrl("/home")
            .and()
            .logout()
                .logoutSuccessUrl("/login?logout");
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin").password(passwordEncoder().encode("password")).roles("ADMIN");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 5.4 테스트 지원 비교

#### 5.4.1 Spring 레거시

```java
// Spring MVC 테스트
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:test-context.xml"})
@WebAppConfiguration
public class UserControllerTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @Before
    public void setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }

    @Test
    public void testGetUser() throws Exception {
        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.name", is("John")));
    }
}
```

#### 5.4.2 Spring Boot

```java
// Spring Boot 테스트
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void testGetUser() throws Exception {
        User user = new User(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(user);

        mockMvc.perform(get("/users/1"))
            .andExpect(status().isOk())
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))
            .andExpect(jsonPath("$.name", is("John")));
    }
}
```

## 6. 사용 사례별 선택 가이드

### 6.1 Spring 레거시가 적합한 경우

- **기존 레거시 시스템 유지보수**
  - 대규모 리팩토링이 어려운 상황
  - 기존 XML 설정에 의존하는 복잡한 시스템
- **특정 엔터프라이즈 환경**
  - 특정 WAS(WebLogic, WebSphere 등)에 종속된 환경
  - 특별한 기업 내부 표준이 있는 경우
- **완전한 커스터마이징 필요**
  - 자동 설정으로 해결할 수 없는 매우 특수한 요구사항
  - 모든 설정을 세밀하게 제어해야 하는 경우

### 6.2 Spring Boot가 적합한 경우

- **새로운 프로젝트 시작**
  - 그린필드(Greenfield) 프로젝트
  - 빠른 개발 시작과 프로토타이핑
- **마이크로서비스 아키텍처**
  - 독립적으로 배포 가능한 서비스 개발
  - 클라우드 환경에서 실행될 애플리케이션
- **DevOps 및 CI/CD 환경**

  - 자동화된 빌드 및 배포 파이프라인
  - 컨테이너 기반 인프라(Docker, Kubernetes)

- **API 중심 애플리케이션**
  - RESTful 서비스 개발
  - 백엔드 API 서버 구축

### 6.3 하이브리드 접근법이 적합한 경우

- **점진적 마이그레이션**
  - 레거시 시스템을 단계적으로 현대화
  - 리스크를 관리하며 전환하는 경우
- **조직적 제약**
  - 기술 스택 전환에 시간이 필요한 경우
  - 팀의 기술 습득 곡선을 고려해야 하는 상황

## 7. 결론 및 미래 전망

### 7.1 결론

Spring Framework(레거시)와 Spring Boot는 모두 자바 기반 애플리케이션 개발을 위한 강력한 도구입니다. Spring Framework는 유연성과 세밀한 제어를 제공하는 반면, Spring Boot는 개발 생산성과 현대적인 애플리케이션 개발 경험을 제공합니다.

프로젝트의 요구사항, 팀의 기술적 배경, 인프라 환경, 그리고 비즈니스 목표에 따라 적절한 프레임워크를 선택하는 것이 중요합니다. 또한, 두 프레임워크는 상호 배타적이지 않으며, 하이브리드 접근법이나 점진적 마이그레이션을 통해 두 프레임워크의 장점을 모두 활용할 수 있습니다.

### 7.2 미래 전망

- **Spring Boot의 지속적인 발전**
  - 더 많은 자동 설정과 스타터 추가
  - GraalVM 네이티브 이미지 지원 강화 (Spring Native)
  - 클라우드 네이티브 기능 확장
- **반응형 프로그래밍의 성장**
  - WebFlux와 Project Reactor의 발전
  - 비동기, 논블로킹 애플리케이션의 표준화
- **클라우드 네이티브 에코시스템 통합**
  - Kubernetes, Service Mesh 기술과의 통합 강화
  - 클라우드 환경에 최적화된 기능 확장
- **Spring 레거시의 지속적인 지원**
  - 엔터프라이즈 환경에서의 안정적인 지원
  - 점진적 현대화 경로 제공

Spring 생태계는 계속해서 발전하고 있으며, 개발자들은 자신의 프로젝트 요구사항에 가장 적합한 접근 방식을 선택하면서도, 새로운 기술과 패턴을 적용할 수 있는 유연성을 가질 수 있습니다.

## 8. 참고 자료

- [Spring Framework 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/)
- [Spring Boot 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring to Spring Boot Migration Guide](https://spring.io/guides/tutorials/spring-boot-migration/)
- [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
