# Nginx 플랫폼 상세 문서

## 1. Nginx 개요

Nginx(엔진엑스)는 고성능 웹 서버, 리버스 프록시, 로드 밸런서, HTTP 캐시로 사용되는 오픈 소스 소프트웨어입니다. 비동기 이벤트 기반 아키텍처를 사용하여 적은 자원으로도 높은 동시성을 처리할 수 있는 것이 특징이며, 현대적인 웹 아키텍처에서 필수적인 구성 요소로 자리 잡고 있습니다.

### 1.1 핵심 특징

- **높은 성능**: 이벤트 기반 비동기 처리 방식으로 수만 개의 동시 연결 처리 가능
- **낮은 메모리 사용량**: 워커 프로세스 기반 아키텍처로 효율적인 메모리 관리
- **리버스 프록시 기능**: 다양한 백엔드 서버로의 요청 전달 및 부하 분산
- **로드 밸런싱**: 여러 서버 간 트래픽 분산으로 성능 최적화 및 가용성 향상
- **HTTP 캐싱**: 정적 및 동적 콘텐츠 캐싱으로 성능 향상
- **SSL/TLS 지원**: HTTPS 연결 처리 및 SSL 종료(termination)
- **URL 재작성**: 요청 URL 변경 및 리다이렉션
- **HTTP/2 및 HTTP/3 지원**: 최신 웹 프로토콜 지원
- **모듈식 구조**: 다양한 모듈을 통한 기능 확장

### 1.2 주요 활용 사례

- **웹 서버**: 정적 파일 서빙 (HTML, CSS, JavaScript, 이미지 등)
- **API 게이트웨이**: 다양한 마이크로서비스 앞단의 단일 진입점
- **리버스 프록시**: 백엔드 서버 보호 및 요청 최적화
- **로드 밸런서**: 여러 서버 간의 트래픽 분산
- **캐싱 서버**: 정적/동적 콘텐츠 캐싱으로 응답 시간 단축
- **SSL 종료**: HTTPS 트래픽 처리 및 암호화/복호화 오프로딩
- **미디어 스트리밍**: 비디오/오디오 스트리밍 최적화
- **WebSocket 프록시**: 실시간 양방향 통신 지원

## 2. Nginx 아키텍처 및 동작 원리

### 2.1 이벤트 기반 아키텍처

Nginx는 아파치의 스레드/프로세스 기반 모델과 달리 이벤트 기반 비동기 아키텍처를 사용합니다. 이 접근 방식의 주요 특징은 다음과 같습니다:

- **마스터 프로세스**: 설정 파일 읽기, 포트 바인딩, 워커 프로세스 생성 및 관리
- **워커 프로세스**: 실제 요청 처리, 각 워커는 단일 스레드로 실행되며 이벤트 루프 활용
- **이벤트 루프**: 비동기 I/O 작업을 통해 단일 스레드로 수천 개의 연결 처리
- **비차단 소켓**: 모든 소켓 작업이 비차단(non-blocking) 방식으로 처리

![Nginx Architecture](https://example.com/nginx-architecture.png)

### 2.2 요청 처리 흐름

1. **연결 수락**: 클라이언트 연결 요청을 리스닝 소켓에서 수락
2. **요청 파싱**: HTTP 요청 헤더 및 본문 파싱
3. **위치 결정**: `location` 블록 기반으로 요청을 어떻게 처리할지 결정
4. **콘텐츠 생성/프록시**: 정적 파일 제공 또는 요청을 백엔드로 프록시
5. **필터 적용**: 응답에 다양한 필터 적용 (헤더 수정, 압축 등)
6. **응답 전송**: 클라이언트에 응답 반환

### 2.3 설정 구조

Nginx 설정은 계층적 구조를 가지며, 다음과 같은 주요 컨텍스트로 구성됩니다:

- **main**: 전역 설정 (워커 프로세스 수, 로깅, PID 파일 위치 등)
- **events**: 연결 처리 관련 설정 (워커 연결 수, 이벤트 모델 등)
- **http**: HTTP 서버 설정
  - **server**: 가상 호스트 정의
    - **location**: URL 패턴별 처리 규칙
- **stream**: TCP/UDP 트래픽 처리 설정
- **mail**: 메일 프록시 설정

## 3. Nginx 도커 구성 분석

현재 프로젝트에서 사용 중인 Nginx 도커 구성을 살펴보겠습니다:

### 3.1 Dockerfile 분석

```dockerfile
FROM alpine:3.18 AS builder

RUN apk add --no-cache \
    gcc libc-dev make pcre-dev openssl-dev zlib-dev curl perl

ENV NGINX_VERSION=1.25.2
ENV NGINX_PURGE_MODULE_VERSION=2.3

WORKDIR /tmp
RUN curl -LO http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
RUN tar -zxvf nginx-${NGINX_VERSION}.tar.gz

RUN curl -LO https://github.com/FRiCKLE/ngx_cache_purge/archive/${NGINX_PURGE_MODULE_VERSION}.tar.gz
RUN tar -zxvf ${NGINX_PURGE_MODULE_VERSION}.tar.gz

WORKDIR /tmp/nginx-${NGINX_VERSION}
RUN ./configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/usr/local/sbin/nginx \
    --with-http_stub_status_module \
    --with-http_ssl_module \
    --with-http_gzip_static_module \
    --with-http_realip_module \
    --with-http_v2_module \
    --add-module=/tmp/ngx_cache_purge-${NGINX_PURGE_MODULE_VERSION}

RUN make && make install

FROM alpine:3.18
RUN apk add --no-cache pcre openssl zlib
RUN addgroup -S nginx && adduser -S -G nginx -h /nonexistent -s /sbin/nologin nginx

COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /usr/local/sbin/nginx /usr/local/sbin/nginx
RUN mkdir -p /var/cache/nginx /var/log/nginx && \
    chown -R nginx:nginx /var/cache/nginx /var/log/nginx

# prod용 nginx.conf (SSL 포함) 복사
COPY docker/prod/nginx/nginx.conf /etc/nginx/nginx.conf

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
```

구성 요소:

- **멀티 스테이지 빌드**:
  - 첫 번째 스테이지: Nginx 소스 컴파일 및 설치
  - 두 번째 스테이지: 필요한 파일만 복사하여 최종 이미지 생성
- **Nginx 버전**: 1.25.2 (최신 안정 버전)
- **ngx_cache_purge 모듈**: 캐시 퍼지 기능을 제공하는 외부 모듈 통합
- **컴파일 옵션**:
  - `--with-http_stub_status_module`: 기본 상태 정보 제공
  - `--with-http_ssl_module`: SSL/TLS 지원
  - `--with-http_gzip_static_module`: 정적 파일 압축
  - `--with-http_realip_module`: 실제 클라이언트 IP 확인
  - `--with-http_v2_module`: HTTP/2 프로토콜 지원
- **최소 이미지**: Alpine Linux 기반으로 경량화

### 3.2 nginx.conf 분석

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 로그 설정
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';
    access_log /dev/stdout main;
    error_log  /dev/stderr warn;

    sendfile        on;
    keepalive_timeout  65;
    server_tokens off;

    # SSL/TLS 설정 시 필요
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    # 캐시 영역 (API 응답용, 필요시)
    proxy_cache_path /var/cache/nginx/products
                     levels=1:2
                     keys_zone=cache_products:10m
                     inactive=10m
                     max_size=1g;

    proxy_cache_path /var/cache/nginx/styles
                     levels=1:2
                     keys_zone=cache_styles:10m
                     inactive=10m
                     max_size=1g;

    proxy_cache_path /var/cache/nginx/es_products
                     levels=1:2
                     keys_zone=cache_es:10m
                     inactive=10m
                     max_size=1g;

    map $query_string $bypass_products_cache {
        default 0;
    }
    map $query_string $bypass_styles_cache {
        ""       0;
        default  1;
    }

    # 80번 포트: HTTP
    server {
        listen 80;
        server_name www.pinjun.xyz;

        # Certbot webroot 인증용
        location /.well-known/acme-challenge/ {
            root /var/www/certbot;
        }

        # 그 외 요청은 HTTPS로 리다이렉트
        location / {
            return 301 https://$host$request_uri;
        }
    }

    # 443번 포트: HTTPS
    server {
        listen 443 ssl;
        server_name www.pinjun.xyz;

        # SSL 인증서(Certbot으로 발급)
        ssl_certificate     /etc/letsencrypt/live/www.pinjun.xyz/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/www.pinjun.xyz/privkey.pem;

        # TLS 설정
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;
        ssl_prefer_server_ciphers on;

        # 정적 파일 서빙
        root /usr/share/nginx/html;
        index index.html;

        # 프론트엔드 프록시
        location / {
            proxy_pass http://front:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        # WebSocket 프록시
        location /api/ws {
            proxy_pass http://app:8080/ws;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
        }

        # API 프록시 (기본)
        location /api/ {
            proxy_pass http://app:8080/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 캐싱된 API 엔드포인트
        location /api/products/query {
            proxy_pass http://app:8080/products/query;
            proxy_cache            cache_products;
            proxy_cache_valid      200 10m;
            proxy_cache_valid      404 1m;
            add_header             X-Cache-Status $upstream_cache_status;
            proxy_cache_key        "$scheme://$host$request_uri";
            proxy_ignore_headers   Cache-Control Expires;
        }

        # 상품 상세 조회 (캐싱 비활성화)
        location ~* ^/api/products/query/([0-9]+)/detail$ {
            rewrite ^/api/products/query/([0-9]+)/detail$ /products/query/$1/detail break;
            proxy_pass http://app:8080;
            proxy_cache_bypass    1;
            proxy_no_cache        1;
        }

        # 캐시 퍼지 엔드포인트
        location ~ /purge_products(/.*) {
            allow all;
            proxy_cache_purge cache_products $scheme$host$1;
        }

        # Kibana 프록시
        location /kibana/ {
            proxy_pass http://kibana:5601/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # STT/TTS 서비스 프록시
        location /talk/ {
            proxy_pass http://stt_tts:5000/;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            client_max_body_size 20M;
        }
    }
}
```

주요 설정:

- **워커 프로세스**: `auto` (CPU 코어 수에 맞게 자동 설정)
- **워커 연결**: 1024개의 동시 연결 처리
- **로깅 설정**: 접근 로그는 stdout으로, 오류 로그는 stderr로 출력
- **보안 설정**: `server_tokens off`로 Nginx 버전 정보 숨김
- **SSL/TLS**: TLSv1.2 및 TLSv1.3만 허용, 강력한 암호화 설정
- **캐시 설정**: 세 개의 별도 캐시 영역 정의 (products, styles, es_products)
- **가상 호스트**:
  - HTTP(80) → HTTPS(443) 리다이렉션
  - Let's Encrypt SSL 인증서 적용
  - 다양한 백엔드 서비스로의 프록시 설정
  - WebSocket 연결 지원
  - 선택적 캐싱 및 캐시 퍼지 기능

### 3.3 docker-compose.yml 설정

```yaml
services:
  nginx:
    build:
      context: ../..
      dockerfile: docker/prod/nginx/Dockerfile-nginx-prod
    container_name: nginx_prod
    ports:
      - "80:80"
      - "443:443"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    environment:
      TZ: Asia/Seoul
    depends_on:
      - mysql
      - redis
      - kafka
      - elasticsearch
      - front
      - app
      - certbot
      - stt_tts
    volumes:
      - letsencrypt_data:/etc/letsencrypt
      - certbot_www:/var/www/certbot
```

주요 설정:

- **포트 매핑**: 80(HTTP)와 443(HTTPS) 포트 노출
- **호스트 추가**: `host.docker.internal`을 호스트 게이트웨이로 설정
- **시간대 설정**: 한국 시간대(Asia/Seoul) 사용
- **의존성 설정**: 다른 서비스에 대한 의존성 정의
- **볼륨 마운트**:
  - `letsencrypt_data`: SSL 인증서 저장
  - `certbot_www`: Let's Encrypt 인증 챌린지 파일 저장

## 4. Nginx 고급 기능 및 최적화

### 4.1 캐싱 전략

#### 4.1.1 캐시 영역 설정

```nginx
proxy_cache_path /var/cache/nginx/data levels=1:2 keys_zone=my_cache:10m
                 inactive=60m max_size=1g;
```

주요 매개변수:

- **levels**: 캐시 파일 디렉토리 구조 (1:2는 2단계 디렉토리)
- **keys_zone**: 캐시 키와 메타데이터 저장 영역 (이름:크기)
- **inactive**: 지정 시간 동안 접근되지 않은 항목 삭제
- **max_size**: 최대 캐시 크기 (초과 시 LRU 방식으로 제거)

#### 4.1.2 캐시 사용 설정

```nginx
location /api/cacheable/ {
    proxy_cache my_cache;
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404     1m;
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    proxy_cache_lock on;
    proxy_cache_background_update on;
    add_header X-Cache-Status $upstream_cache_status;

    proxy_pass http://backend;
}
```

주요 설정:

- **proxy_cache_valid**: 상태 코드별 캐시 유효 시간
- **proxy_cache_use_stale**: 백엔드 오류 시 오래된 캐시 사용
- **proxy_cache_lock**: 동일 리소스 동시 요청 시 하나만 백엔드로 전달
- **proxy_cache_background_update**: 백그라운드에서 만료된 캐시 갱신
- **X-Cache-Status**: 캐시 상태 헤더 추가 (디버깅용)

#### 4.1.3 캐시 무효화(Cache Purging)

```nginx
# 캐시 퍼지 모듈 설정
location ~ /purge(/.*) {
    allow 127.0.0.1;
    allow 192.168.0.0/24;
    deny all;

    proxy_cache_purge my_cache $scheme$host$1;
}
```

### 4.2 로드 밸런싱

#### 4.2.1 업스트림 서버 그룹 설정

```nginx
upstream backend {
    least_conn;
    server backend1.example.com weight=3;
    server backend2.example.com weight=2;
    server backend3.example.com weight=1 backup;
    server backend4.example.com max_fails=3 fail_timeout=30s;

    keepalive 32;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

로드 밸런싱 알고리즘:

- **round-robin**: 기본값, 요청을 순차적으로 분배
- **least_conn**: 활성 연결이 가장 적은 서버로 라우팅
- **ip_hash**: 클라이언트 IP를 기반으로 일관된 서버 선택
- **hash**: 지정된 키(URL, 쿠키 등)를 기반으로 일관된 서버 선택

주요 매개변수:

- **weight**: 서버 가중치 (높을수록 더 많은 요청 처리)
- **backup**: 주 서버가 모두 실패할 때만 사용
- **max_fails** & **fail_timeout**: 장애 감지 및 복구 설정
- **keepalive**: 백엔드 서버와의 유지 연결 수

#### 4.2.2 헬스 체크

```nginx
# 능동적 헬스 체크 (Nginx Plus 기능)
upstream backend {
    zone backend 64k;

    server backend1.example.com:8080;
    server backend2.example.com:8080;

    health_check interval=5s passes=3 fails=2;
}

# 수동적 헬스 체크 (오픈 소스 Nginx)
upstream backend {
    server backend1.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 max_fails=3 fail_timeout=30s;
}
```

### 4.3 SSL/TLS 최적화

#### 4.3.1 강력한 보안 설정

```nginx
# SSL/TLS 설정
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
ssl_ecdh_curve secp384r1;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

주요 설정:

- **최신 프로토콜**: TLSv1.2 및 TLSv1.3만 허용
- **강력한 암호화 알고리즘**: 안전한 암호 스위트 지정
- **OCSP 스테이플링**: 인증서 유효성 검증 속도 향상
- **세션 캐싱**: SSL 핸드셰이크 오버헤드 감소
- **보안 헤더**: 다양한 보안 헤더 추가

#### 4.3.2 Let's Encrypt 인증서 자동화

```nginx
server {
    listen 80;
    server_name example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # 기타 SSL 설정...

    # 인증서 자동 갱신을 위한 certbot 설정
    location ~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}
```

### 4.4 성능 최적화 설정

#### 4.4.1 워커 프로세스 및 연결 튜닝

```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections 65535;
    multi_accept on;
}

http {
    keepalive_timeout 65;
    keepalive_requests 1000;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    # 파일 디스크립터 캐싱
    open_file_cache max=200000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
}
```

주요 설정:

- **worker_processes**: CPU 코어 수에 맞게 자동 설정
- **worker_rlimit_nofile**: 각 워커가 열 수 있는 최대 파일 수
- **epoll**: 리눅스에서 고성능 이벤트 처리 메커니즘
- **multi_accept**: 한 번의 이벤트 처리로 여러 연결 수락
- **sendfile**: 커널 공간에서 직접 파일 전송으로 성능 향상
- **tcp_nopush/tcp_nodelay**: TCP 패킷 최적화
- **open_file_cache**: 자주 사용하는 파일 디스크립터 캐싱

#### 4.4.2 Gzip 압축

```nginx
gzip on;
gzip_comp_level 5;
gzip_min_length 256;
gzip_proxied any;
gzip_vary on;
gzip_types
    application/atom+xml
    application/javascript
    application/json
    application/ld+json
    application/manifest+json
    application/rss+xml
    application/vnd.geo+json
    application/vnd.ms-fontobject
    application/x-font-ttf
    application/x-web-app-manifest+json
    application/xhtml+xml
    application/xml
    font/opentype
    image/bmp
    image/svg+xml
    image/x-icon
    text/cache-manifest
    text/css
    text/plain
    text/vcard
    text/vnd.rim.location.xloc
    text/vtt
    text/x-component
    text/x-cross-domain-policy;
```

#### 4.4.3 버퍼 크기 최적화

```nginx
client_max_body_size 50m;
client_body_buffer_size 128k;
client_header_buffer_size 1k;
large_client_header_buffers 4 8k;
output_buffers 2 32k;
fastcgi_buffers 8 16k;
fastcgi_buffer_size 32k;
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;
```

## 5. 캐시 퍼지(Cache Purge) 모듈 활용

### 5.1 ngx_cache_purge 모듈 개요

ngx_cache_purge는 Nginx 캐시 항목을 선택적으로 삭제할 수 있게 해주는 타사 모듈입니다. 이 모듈은 캐시된 콘텐츠를 업데이트할 때 유용하며, 특히 REST API 응답 캐싱에서 중요한 역할을 합니다.

### 5.2 캐시 퍼지 설정 및 사용법

#### 5.2.1 기본 설정

```nginx
# 캐시 영역 정의
proxy_cache_path /var/cache/nginx/cache levels=1:2 keys_zone=content_cache:10m
                 inactive=1d max_size=1g;

server {
    # 캐시 퍼지 위치 블록
    location ~ /purge(/.*) {
        # 허용된 IP만 접근 가능
        allow 127.0.0.1;
        allow 10.0.0.0/8;
        deny all;

        # 캐시 키 구성 ($1은 정규식에서 캡처된 경로)
        proxy_cache_purge content_cache $scheme$host$1;
    }

    # 캐시되는 콘텐츠 위치 블록
    location / {
        proxy_cache content_cache;
        proxy_cache_valid 200 1h;
        proxy_cache_key $scheme$host$request_uri;

        proxy_pass http://backend;
    }
}
```

#### 5.2.2 캐시 퍼지 요청 예시

```bash
# 특정 URL의 캐시 항목 삭제
curl -X PURGE http://example.com/purge/api/products/1

# 와일드카드 패턴 매칭을 통한 캐시 항목 삭제 (특정 구현에 따라 다름)
curl -X PURGE http://example.com/purge/api/products*
```

### 5.3 캐시 관리 자동화

#### 5.3.1 캐시 전략 구현

```
1. 일반적인 읽기 요청은 캐시에서 제공
2. 데이터 변경이 발생하면 관련 캐시 항목 퍼지
3. 다음 요청 시 새로운 데이터로 캐시 재생성
```

#### 5.3.2 API 백엔드와의 통합

```java
// Spring Boot 서비스에서 캐시 무효화 구현 예시
@Service
public class CacheInvalidationService {

    private final RestTemplate restTemplate = new RestTemplate();
    private final String cacheServerUrl = "http://nginx:80";

    // 제품 캐시 무효화
    public void invalidateProductCache(Long productId) {
        String purgeUrl = cacheServerUrl + "/purge/api/products/" + productId;
        HttpHeaders headers = new HttpHeaders();
        headers.set("Host", "www.example.com");

        try {
            HttpEntity<String> entity = new HttpEntity<>(headers);
            restTemplate.exchange(purgeUrl, HttpMethod.PURGE, entity, String.class);
            log.info("Cache purged for product ID: {}", productId);
        } catch (Exception e) {
            log.error("Failed to purge cache for product ID: {}", productId, e);
        }
    }

    // 카테고리별 제품 캐시 무효화
    public void invalidateCategoryProductsCache(String category) {
        String purgeUrl = cacheServerUrl + "/purge/api/products?category=" + category;
        HttpHeaders headers = new HttpHeaders();
        headers.set("Host", "www.example.com");

        try {
            HttpEntity<String> entity = new HttpEntity<>(headers);
            restTemplate.exchange(purgeUrl, HttpMethod.PURGE, entity, String.class);
            log.info("Cache purged for category: {}", category);
        } catch (Exception e) {
            log.error("Failed to purge cache for category: {}", category, e);
        }
    }
}
```

#### 5.3.3 주기적 캐시 정리

```bash
#!/bin/sh
# /scripts/clear_caches.sh

# Elasticsearch 캐시 퍼지
curl -X PURGE http://nginx/purge_es/api/es/products?query=popular

# 3시간 이상 지난 제품 캐시 퍼지
find /var/cache/nginx/products -type f -mmin +180 -delete

# Redis 캐시 플러시
redis-cli -h redis flushdb async
```

## 6. Nginx 활용 사례

### 6.1 API 게이트웨이 구현

#### 6.1.1 마이크로서비스 라우팅

```nginx
# API 게이트웨이 설정
http {
    # 업스트림 정의
    upstream user_service {
        server user-service:8080;
    }

    upstream product_service {
        server product-service:8080;
    }

    upstream order_service {
        server order-service:8080;
    }

    # API 게이트웨이 서버
    server {
        listen 80;
        server_name api.example.com;

        # 사용자 서비스 라우팅
        location /api/users {
            proxy_pass http://user_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Request-ID $request_id;
        }

        # 제품 서비스 라우팅
        location /api/products {
            proxy_pass http://product_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Request-ID $request_id;
        }

        # 주문 서비스 라우팅
        location /api/orders {
            proxy_pass http://order_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Request-ID $request_id;
        }

        # API 문서
        location /api/docs {
            proxy_pass http://swagger-ui:8080;
        }
    }
}
```

#### 6.1.2 CORS 설정

```nginx
# CORS 허용 설정
location /api/ {
    # CORS 전처리 요청(preflight) 처리
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
    }

    # 일반 요청에 대한 CORS 헤더
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

    proxy_pass http://api_backend;
}
```

#### 6.1.3 인증 및 권한 검증

```nginx
# JWT 인증 미들웨어
location /api/ {
    # JWT 검증을 위한 auth_request 모듈 사용
    auth_request /auth/validate;

    # 원본 URI를 auth 서비스로 전달
    auth_request_set $auth_status $upstream_status;

    # 인증 서비스에서 추출한 사용자 ID를 백엔드로 전달
    auth_request_set $user_id $upstream_http_x_user_id;
    proxy_set_header X-User-ID $user_id;

    # 백엔드 서비스로 요청 전달
    proxy_pass http://api_backend;
}

# 인증 검증 내부 위치
location = /auth/validate {
    internal;
    proxy_pass http://auth-service:8080/validate;
    proxy_pass_request_body off;
    proxy_set_header Content-Length "";
    proxy_set_header X-Original-URI $request_uri;
    proxy_set_header X-Original-Method $request_method;
    proxy_set_header Authorization $http_authorization;
}
```

### 6.2 정적 콘텐츠 최적화

#### 6.2.1 정적 자산 서빙

```nginx
# 정적 파일 서빙 최적화
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
    root /var/www/static;
    expires 30d;
    add_header Cache-Control "public, max-age=2592000";
    add_header Vary Accept-Encoding;
    access_log off;

    # 정적 파일 최적화
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    # gzip 압축
    gzip on;
    gzip_min_length 1000;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml;
}
```

#### 6.2.2 브라우저 캐싱 최적화

```nginx
# 버전 기반 캐싱 (URL에 버전 포함)
location ~* ^/assets/v[0-9]+/ {
    root /var/www/static;
    expires max;
    add_header Cache-Control "public, immutable";
    access_log off;
}

# HTML 파일 (짧은 캐시)
location ~* \.html$ {
    root /var/www/html;
    expires 10m;
    add_header Cache-Control "public, must-revalidate";
}
```

#### 6.2.3 SPA(Single Page Application) 설정

```nginx
# React/Vue/Angular 등의 SPA 설정
location / {
    root /var/www/html;
    index index.html;
    try_files $uri $uri/ /index.html;

    # 캐싱 설정
    location ~* \.(?:css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # 소스맵 접근 제한
    location ~* \.map$ {
        deny all;
    }
}
```

### 6.3 실시간 통신 지원

#### 6.3.1 WebSocket 프록시

```nginx
# WebSocket 요청을 백엔드로 프록시
location /ws/ {
    proxy_pass http://websocket_backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # WebSocket 타임아웃 설정
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;

    # 긴 연결이 프록시 캐시나 다른 중간 장치에 의해 닫히지 않도록 설정
    proxy_buffering off;
}
```

#### 6.3.2 Server-Sent Events (SSE) 지원

```nginx
# Server-Sent Events 지원
location /events/ {
    proxy_pass http://sse_backend;
    proxy_http_version 1.1;

    # 청크 전송 인코딩 유지
    proxy_set_header Connection "";

    # 버퍼링 비활성화
    proxy_buffering off;

    # 타임아웃 설정
    proxy_read_timeout 24h;

    # 헤더 설정
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

### 6.4 보안 강화

#### 6.4.1 기본 보안 헤더

```nginx
# 공통 보안 헤더
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header X-Frame-Options SAMEORIGIN;
add_header Referrer-Policy strict-origin-when-cross-origin;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data:; connect-src 'self' https://api.example.com";
```

#### 6.4.2 HTTPS 강제 및 HSTS

```nginx
# HTTP에서 HTTPS로 리디렉션
server {
    listen 80;
    server_name example.com www.example.com;

    # HTTP 요청을 HTTPS로 리디렉션
    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS 서버
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL 인증서 설정
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL 파라미터 최적화
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384";
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # HSTS (HTTP Strict Transport Security) 설정
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
}
```

#### 6.4.3 요율 제한(Rate Limiting)

```nginx
# IP 기반 요율 제한
http {
    # 요율 제한 영역 정의
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    server {
        # API 요청에 요율 제한 적용
        location /api/ {
            # 10 RPS 제한, 최대 20개 요청 버스트 허용
            limit_req zone=api_limit burst=20 nodelay;

            # 제한 초과 시 커스텀 오류 페이지
            limit_req_status 429;
            error_page 429 = @rate_limited;

            proxy_pass http://api_backend;
        }

        # 요율 제한 초과 처리
        location @rate_limited {
            add_header Content-Type application/json;
            add_header Retry-After 60;
            return 429 '{"error":"Too many requests, please try again later."}';
        }
    }
}
```

## 7. 문제 해결 및 디버깅 가이드

### 7.1 일반적인 문제와 해결책

#### 7.1.1 502 Bad Gateway

- **증상**: 브라우저가 502 Bad Gateway 오류를 표시
- **가능한 원인**:
  - 백엔드 서버가 응답하지 않음
  - 프록시 설정 오류
  - 백엔드 연결 시간 초과
- **해결책**:
  - 백엔드 서버 상태 확인
  - `proxy_read_timeout`, `proxy_connect_timeout` 값 증가
  - 로그 파일 확인 (`/var/log/nginx/error.log`)
  - 백엔드 호스트 이름 및 포트 설정 확인

#### 7.1.2 404 Not Found

- **증상**: 브라우저가 404 Not Found 오류를 표시
- **가능한 원인**:
  - 잘못된 루트 디렉토리 설정
  - 파일 경로 오류
  - 위치 블록 설정 오류
- **해결책**:
  - `root` 및 `alias` 디렉티브 확인
  - 파일 권한 및 소유권 확인
  - `try_files` 디렉티브 확인

#### 7.1.3 성능 문제

- **증상**: 응답 시간 증가, 높은 지연 시간
- **가능한 원인**:
  - 부적절한 워커 프로세스 수
  - 부족한 워커 연결 수
  - 캐싱 설정 부족
  - 백엔드 서버 병목 현상
- **해결책**:
  - `worker_processes` 및 `worker_connections` 최적화
  - 정적 콘텐츠 캐싱 활성화
  - Gzip 압축 설정
  - 로드 밸런싱 구현

### 7.2 디버깅 도구 및 명령

#### 7.2.1 로그 분석

```bash
# 실시간 오류 로그 모니터링
tail -f /var/log/nginx/error.log

# 접근 로그에서 오류 상태 코드 검색
grep -E ' (4|5)[0-9][0-9] ' /var/log/nginx/access.log

# 특정 IP의 요청 추적
grep 192.168.1.1 /var/log/nginx/access.log

# 요청 처리 시간이 느린 요청 찾기
awk '$NF > 1 {print $0}' /var/log/nginx/access.log
```

#### 7.2.2 구성 테스트 및 확인

```bash
# 구성 파일 문법 검사
nginx -t

# 모든 설정 덤프 (실제 실행 중인 설정)
nginx -T

# 설정 파일 디버깅 모드로 실행
nginx -t -c /etc/nginx/nginx.conf -v
```

#### 7.2.3 상태 모니터링

```nginx
# Nginx 상태 모니터링 위치 블록
location /nginx_status {
    stub_status on;
    allow 127.0.0.1;
    deny all;
}
```

```bash
# 상태 확인
curl http://localhost/nginx_status

# 출력 예시:
# Active connections: 43
# server accepts handled requests
# 97450 97450 139639
# Reading: 0 Writing: 5 Waiting: 38
```

### 7.3 Docker 환경에서의 디버깅

#### 7.3.1 로그 확인

```bash
# Nginx 컨테이너 로그 확인
docker logs nginx_prod

# 실시간 로그 추적
docker logs -f nginx_prod

# 최근 100줄만 표시
docker logs --tail 100 nginx_prod
```

#### 7.3.2 컨테이너 내부 접속

```bash
# Nginx 컨테이너에 접속
docker exec -it nginx_prod sh

# 구성 확인
docker exec -it nginx_prod nginx -T

# 프로세스 확인
docker exec -it nginx_prod ps aux
```

#### 7.3.3 네트워크 디버깅

```bash
# 컨테이너 네트워크 확인
docker network inspect bridge

# 컨테이너 내에서 네트워크 테스트
docker exec -it nginx_prod ping app

# 포트 바인딩 확인
docker port nginx_prod

# netstat로 리스닝 포트 확인
docker exec -it nginx_prod netstat -tulpn
```

## 8. Nginx와 다른 서비스 통합

### 8.1 React 프론트엔드 서빙

```nginx
# React SPA 서빙 설정
server {
    listen 80;
    server_name myapp.com;

    root /usr/share/nginx/html;
    index index.html;

    # 모든 경로를 index.html로 리디렉션 (React 라우터용)
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 정적 자산 캐싱
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 소스맵 보호
    location ~* \.map$ {
        deny all;
    }
}
```

### 8.2 Spring Boot 앱 프록시

```nginx
# Spring Boot API 프록시 설정
server {
    listen 80;
    server_name api.myapp.com;

    # API 요청 프록시
    location / {
        proxy_pass http://app:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket 엔드포인트
    location /ws {
        proxy_pass http://app:8080/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 3600s;
    }
}
```

### 8.3 Kibana와 같은 내부 서비스 프록시

```nginx
# Kibana 서비스 프록시
server {
    listen 80;
    server_name kibana.internal.com;

    # 기본 인증 추가
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;

    # 특정 IP만 접근 허용
    allow 10.0.0.0/8;
    allow 192.168.0.0/16;
    deny all;

    # Kibana 프록시
    location / {
        proxy_pass http://kibana:5601;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 서브 경로에서 실행 시
        proxy_set_header basePath "/kibana";
        rewrite ^/kibana/(.*)$ /$1 break;
    }
}
```

## 9. 참고 자료 및 추가 학습 리소스

### 9.1 공식 문서

- [Nginx 공식 문서](https://nginx.org/en/docs/)
- [Nginx Admin Guide](https://docs.nginx.com/nginx/admin-guide/)
- [Nginx Docker 이미지](https://hub.docker.com/_/nginx)
- [ngx_cache_purge 모듈](https://github.com/FRiCKLE/ngx_cache_purge)

### 9.2 책 추천

- "Nginx HTTP Server" by Clément Nedelcu
- "Nginx Cookbook" by Derek DeJonghe
- "Mastering Nginx" by Dimitri Aivaliotis

### 9.3 온라인 강좌 및 튜토리얼

- [Nginx Fundamentals](https://www.udemy.com/course/nginx-fundamentals/)
- [Advanced Nginx Configuration](https://www.digitalocean.com/community/tutorials/advanced-nginx-configuration-guide)
- [Nginx 보안 강화 가이드](https://www.acunetix.com/blog/web-security-zone/hardening-nginx/)# Nginx 플랫폼 상세 문서

## 1. Nginx 개요

Nginx(엔진엑스)는 고성능 웹 서버, 리버스 프록시, 로드 밸런서, HTTP 캐시로 사용되는 오픈 소스 소프트웨어입니다. 비동기 이벤트 기반 아키텍처를 사용하여 적은 자원으로도 높은 동시성을 처리할 수 있는 것이 특징이며, 현대적인 웹 아키텍처에서 필수적인 구성 요소로 자리 잡고 있습니다.

### 1.1 핵심 특징

- **높은 성능**: 이벤트 기반 비동기 처리 방식으로 수만 개의 동시 연결 처리 가능
- **낮은 메모리 사용량**: 워커 프로세스 기반 아키텍처로 효율적인 메모리 관리
- **리버스 프록시 기능**: 다양한 백엔드 서버로의 요청 전달 및 부하 분산
- **로드 밸런싱**: 여러 서버 간 트래픽 분산으로 성능 최적화 및 가용성 향상
- **HTTP 캐싱**: 정적 및 동적 콘텐츠 캐싱으로 성능 향상
- **SSL/TLS 지원**: HTTPS 연결 처리 및 SSL 종료(termination)
- **URL 재작성**: 요청 URL 변경 및 리다이렉션
- **HTTP/2 및 HTTP/3 지원**: 최신 웹 프로토콜 지원
- **모듈식 구조**: 다양한 모듈을 통한 기능 확장

### 1.2 주요 활용 사례

- **웹 서버**: 정적 파일 서빙 (HTML, CSS, JavaScript, 이미지 등)
- **API 게이트웨이**: 다양한 마이크로서비스 앞단의 단일 진입점
- **리버스 프록시**: 백엔드 서버 보호 및 요청 최적화
- **로드 밸런서**: 여러 서버 간의 트래픽 분산
- **캐싱 서버**: 정적/동적 콘텐츠 캐싱으로 응답 시간 단축
- **SSL 종료**: HTTPS 트래픽 처리 및 암호화/복호화 오프로딩
- **미디어 스트리밍**: 비디오/오디오 스트리밍 최적화
- **WebSocket 프록시**: 실시간 양방향 통신 지원

## 2. Nginx 아키텍처 및 동작 원리

### 2.1 이벤트 기반 아키텍처

Nginx는 아파치의 스레드/프로세스 기반 모델과 달리 이벤트 기반 비동기 아키텍처를 사용합니다. 이 접근 방식의 주요 특징은 다음과 같습니다:

- **마스터 프로세스**: 설정 파일 읽기, 포트 바인딩, 워커 프로세스 생성 및 관리
- **워커 프로세스**: 실제 요청 처리, 각 워커는 단일 스레드로 실행되며 이벤트 루프 활용
- **이벤트 루프**: 비동기 I/O 작업을 통해 단일 스레드로 수천 개의 연결 처리
- **비차단 소켓**: 모든 소켓 작업이 비차단(non-blocking) 방식으로 처리

![Nginx Architecture](https://example.com/nginx-architecture.png)

### 2.2 요청 처리 흐름

1. **연결 수락**: 클라이언트 연결 요청을 리스닝 소켓에서 수락
2. **요청 파싱**: HTTP 요청 헤더 및 본문 파싱
3. **위치 결정**: `location` 블록 기반으로 요청을 어떻게 처리할지 결정
4. **콘텐츠 생성/프록시**: 정적 파일 제공 또는 요청을 백엔드로 프록시
5. **필터 적용**: 응답에 다양한 필터 적용 (헤더 수정, 압축 등)
6. **응답 전송**: 클라이언트에 응답 반환

### 2.3 설정 구조

Nginx 설정은 계층적 구조를 가지며, 다음과 같은 주요 컨텍스트로 구성됩니다:

- **main**: 전역 설정 (워커 프로세스 수, 로깅, PID 파일 위치 등)
- **events**: 연결 처리 관련 설정 (워커 연결 수, 이벤트 모델 등)
- **http**: HTTP 서버 설정
  - **server**: 가상 호스트 정의
    - **location**: URL 패턴별 처리 규칙
- **stream**: TCP/UDP 트래픽 처리 설정
- **mail**: 메일 프록시 설정

## 3. Nginx 도커 구성 분석

현재 프로젝트에서 사용 중인 Nginx 도커 구성을 살펴보겠습니다:

### 3.1 Dockerfile 분석

```dockerfile
FROM alpine:3.18 AS builder

RUN apk add --no-cache \
    gcc libc-dev make pcre-dev openssl-dev zlib-dev curl perl

ENV NGINX_VERSION=1.25.2
ENV NGINX_PURGE_MODULE_VERSION=2.3

WORKDIR /tmp
RUN curl -LO http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
RUN tar -zxvf nginx-${NGINX_VERSION}.tar.gz

RUN curl -LO https://github.com/FRiCKLE/ngx_cache_purge/archive/${NGINX_PURGE_MODULE_VERSION}.tar.gz
RUN tar -zxvf ${NGINX_PURGE_MODULE_VERSION}.tar.gz

WORKDIR /tmp/nginx-${NGINX_VERSION}
RUN ./configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/usr/local/sbin/nginx \
    --with-http_stub_status_module \
    --with-http_ssl_module \
    --with-http_gzip_static_module \
    --with-http_realip_module \
    --with-http
```
