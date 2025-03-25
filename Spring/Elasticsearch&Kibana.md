# Elasticsearch & Kibana 플랫폼 상세 문서

## 1. Elasticsearch 개요

Elasticsearch는 Lucene 기반의 분산형 RESTful 검색 및 분석 엔진으로, 대용량 데이터를 빠르게 저장, 검색, 분석할 수 있게 해주는 오픈 소스 도구입니다. 실시간 데이터 처리, 전문 검색(full-text search), 로그 분석, 메트릭 저장 등 다양한 사용 사례에 적합합니다.

### 1.1 핵심 특징

- **분산 아키텍처**: 수평적 확장성을 제공하는 분산 시스템
- **실시간 검색 및 분석**: 데이터 색인화 즉시 검색 가능
- **다양한 데이터 타입 지원**: 정형, 비정형, 지리적 데이터 등 지원
- **RESTful API**: HTTP를 통한 간단한 API 호출 방식
- **스키마리스**: 필드를 사전에 정의할 필요 없이 문서 추가 가능
- **고급 쿼리 언어**: 강력하고 유연한 JSON 기반 쿼리 언어
- **집계(Aggregations)**: 데이터 분석과 통계 기능 제공
- **멀티테넌시**: 단일 클러스터에서 여러 인덱스 격리 관리
- **플러그인 아키텍처**: 다양한 확장 기능 제공

### 1.2 주요 활용 사례

- **엔터프라이즈 검색**: 웹사이트, 앱, 문서 등의 검색 기능 구현
- **로그 및 이벤트 데이터 분석**: 시스템 로그, 애플리케이션 로그, 보안 이벤트 분석
- **메트릭 분석**: 시스템 및 비즈니스 메트릭 저장 및 분석
- **자연어 처리**: 텍스트 분석, 감정 분석, 문서 분류 등
- **보안 분석**: 이상 행동 탐지, 보안 이벤트 분석
- **비즈니스 인텔리전스**: 대시보드 및 보고서용 데이터 제공
- **지리공간 분석**: 위치 기반 검색 및 분석

## 2. Kibana 개요

Kibana는 Elasticsearch를 위한 오픈 소스 시각화 및 분석 플랫폼으로, Elasticsearch에 저장된 데이터를 탐색, 시각화, 대시보드화하는 도구입니다. Elastic Stack(이전의 ELK Stack)의 핵심 구성 요소 중 하나입니다.

### 2.1 핵심 특징

- **데이터 탐색**: Elasticsearch 인덱스의 데이터를 쉽게 탐색
- **시각화 도구**: 차트, 그래프, 맵 등 다양한 시각화 생성
- **대시보드**: 여러 시각화를 조합한 대시보드 구성
- **Canvas**: 프레젠테이션 품질의 동적 시각화 작성
- **Lens**: 드래그 앤 드롭 방식의 시각화 도구
- **Timelion**: 시계열 데이터 시각화 특화 기능
- **Dev Tools**: Elasticsearch API와 상호작용하는 개발자 도구
- **마크다운 지원**: 대시보드에 설명 추가 가능
- **공유 및 내보내기**: 대시보드와 시각화 공유 및 PDF 내보내기

### 2.2 주요 활용 사례

- **로그 모니터링**: 애플리케이션 로그 실시간 분석
- **인프라 모니터링**: 서버, 네트워크, 컨테이너 메트릭 관찰
- **비즈니스 인텔리전스**: 데이터 기반 의사결정 지원
- **보안 분석**: 보안 이벤트 시각화 및 이상 징후 탐지
- **지리공간 분석**: 위치 데이터 시각화
- **APM(Application Performance Monitoring)**: 애플리케이션 성능 모니터링

## 3. Elasticsearch 아키텍처 및 동작 원리

### 3.1 핵심 개념

#### 3.1.1 노드(Node)

Elasticsearch의 단일 서버 인스턴스로, 데이터를 저장하고 클러스터의 색인화 및 검색 기능에 참여합니다.

**노드 유형**:

- **마스터 노드(Master Node)**: 클러스터 상태 관리 및 조정
- **데이터 노드(Data Node)**: 데이터 저장 및 CRUD 작업, 검색, 집계 수행
- **인제스트 노드(Ingest Node)**: 인덱싱 전 데이터 전처리 수행
- **코디네이팅 노드(Coordinating Node)**: 클라이언트 요청 라우팅, 병합 등 담당
- **머신러닝 노드(ML Node)**: 머신러닝 작업 수행

#### 3.1.2 클러스터(Cluster)

하나 이상의 노드가 모여 이루는 집합으로, 모든 데이터를 공동으로 보유하고 모든 노드에서 통합 인덱싱 및 검색 기능을 제공합니다.

#### 3.1.3 인덱스(Index)

비슷한 특성을 가진 문서들의 모음으로, 관계형 데이터베이스의 테이블과 유사한 개념입니다. 인덱스는 역색인(inverted index)을 사용하여 빠른 검색을 가능하게 합니다.

#### 3.1.4 문서(Document)

정보를 저장하는 기본 단위로, JSON 형식으로 표현됩니다. 관계형 데이터베이스의 행(row)과 유사합니다.

#### 3.1.5 샤드(Shard)

인덱스를 여러 조각으로 나눈 것으로, 수평적 확장과 병렬 처리를 가능하게 합니다. 샤드는 자체적인 완전한 Lucene 인덱스입니다.

- **프라이머리 샤드(Primary Shard)**: 각 문서가 최초로 저장되는 샤드
- **레플리카 샤드(Replica Shard)**: 프라이머리 샤드의 복제본으로 고가용성 및 읽기 성능 향상

#### 3.1.6 매핑(Mapping)

문서와 그 필드가 저장되고 인덱싱되는 방법을 정의하는 스키마 정의입니다. 관계형 데이터베이스의 테이블 스키마와 유사합니다.

### 3.2 데이터 처리 흐름

#### 3.2.1 인덱싱(Indexing) 과정

1. **문서 제출**: 클라이언트가 `POST` 또는 `PUT` 요청으로 문서 제출
2. **라우팅**: 코디네이팅 노드가 문서의 ID를 기반으로 적절한 샤드로 라우팅
3. **저장**: 프라이머리 샤드가 문서를 저장
4. **복제**: 성공 시 레플리카 샤드에 복제
5. **응답**: 클라이언트에 확인 응답 반환

#### 3.2.2 검색(Search) 과정

1. **쿼리 제출**: 클라이언트가 `GET` 또는 `POST` 요청으로 검색 쿼리 제출
2. **쿼리 파싱**: 코디네이팅 노드가 쿼리 파싱 및 필요한 샤드 결정
3. **샤드 검색**: 각 관련 샤드(프라이머리 또는 레플리카)에 검색 요청 분배
4. **내부 검색**: 각 샤드가 Lucene을 사용하여 로컬 검색 수행
5. **결과 수집**: 코디네이팅 노드가 샤드별 결과 수집 및 병합
6. **결과 반환**: 통합된 결과를 클라이언트에 반환

#### 3.2.3 분석(Analysis) 과정

1. **토큰화(Tokenization)**: 텍스트를 개별 토큰(단어)으로 분리
2. **토큰 필터링**: 토큰 변환, 추가, 제거 등의 처리
3. **텀 생성**: 최종 인덱스 텀 생성
4. **역색인(Inverted Index) 구성**: 텀에서 문서로의 매핑 생성

### 3.3 역색인(Inverted Index) 구조

Elasticsearch의 핵심 검색 효율성은 역색인 구조에서 비롯됩니다:

1. **용어 사전(Term Dictionary)**: 모든 고유 텀(단어)의 정렬된 목록
2. **포스팅 목록(Postings List)**: 각 텀이 나타나는 문서 ID 목록
3. **텀 빈도(Term Frequency)**: 각 문서 내 텀 출현 빈도
4. **위치 정보(Position Information)**: 텀의 문서 내 위치 (구문 쿼리용)
5. **오프셋 정보(Offset Information)**: 텀의 시작/끝 위치 (하이라이팅용)

예시 역색인:

```
Term      | Document IDs
----------|-------------
apple     | 1, 4, 7
banana    | 2, 5, 7
cherry    | 3, 5, 6
```

## 4. Elasticsearch와 Kibana 도커 구성 분석

현재 프로젝트에서 사용 중인 Elasticsearch와 Kibana 도커 구성을 살펴보겠습니다:

### 4.1 Elasticsearch Dockerfile 분석

```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:8.13.4

# nori 플러그인 설치
RUN elasticsearch-plugin install analysis-nori

# --- 필요하다면 root 권한으로 전환 ---
USER root

# 디렉토리 권한 변경
RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/logs \
    && chmod -R 775 /usr/share/elasticsearch/logs

# --- 다시 elasticsearch 유저로 전환 ---
USER elasticsearch

# 사용자 사전, synonyms COPY
RUN mkdir -p /usr/share/elasticsearch/config/analysis
COPY ./docker/elasticsearch/userdict_ko.txt /usr/share/elasticsearch/config/analysis/userdict_ko.txt
COPY ./docker/elasticsearch/synonyms.txt /usr/share/elasticsearch/config/analysis/synonyms.txt

# 단일노드 모드, 보안 비활성
ENV discovery.type=single-node
ENV xpack.security.enabled=false
ENV ES_JAVA_OPTS "-Dfile.encoding=UTF-8 -Dclient.encoding.override=UTF-8"

# 추가 JVM 설정이 필요하면 ENV ES_JAVA_OPTS="-Xms512m -Xmx512m" 등 지정
```

구성 요소:

- **베이스 이미지**: Elasticsearch 8.13.4 공식 이미지 사용
- **플러그인 설치**: 한국어 형태소 분석기 Nori 플러그인 설치
- **권한 설정**: 로그 디렉토리 권한 조정
- **사용자 사전**: 한국어 사용자 사전 및 동의어 사전 추가
- **환경 변수**:
  - `discovery.type=single-node`: 단일 노드 모드로 실행
  - `xpack.security.enabled=false`: 보안 기능 비활성화 (개발 환경용)
  - `ES_JAVA_OPTS`: 자바 인코딩 설정

### 4.2 Kibana Dockerfile 분석

```dockerfile
FROM docker.elastic.co/kibana/kibana:8.10.2

# 기본 Elasticsearch 연결
ENV ELASTICSEARCH_HOSTS=http://elasticsearch:9200

# (중요) Kibana를 /kibana/ 경로에서 서빙하기 위한 설정
ENV SERVER_BASEPATH=/kibana
ENV SERVER_REWRITEBASEPATH=true

EXPOSE 5601

CMD ["/usr/local/bin/kibana-docker"]
```

구성 요소:

- **베이스 이미지**: Kibana 8.10.2 공식 이미지 사용
- **환경 변수**:
  - `ELASTICSEARCH_HOSTS`: Elasticsearch 연결 URL
  - `SERVER_BASEPATH`: Kibana가 동작할 기본 경로 설정
  - `SERVER_REWRITEBASEPATH`: URL 경로 재작성 활성화
- **포트 노출**: 기본 Kibana 포트인 5601 노출
- **실행 명령**: Kibana 도커 스크립트 실행

### 4.3 docker-compose.yml 설정

```yaml
services:
  elasticsearch:
    build:
      context: ../..
      dockerfile: docker/elasticsearch/Dockerfile-elasticsearch
    container_name: es_prod
    environment:
      TZ: Asia/Seoul
      discovery.type: single-node
      xpack.security.enabled: "false"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es_data_prod:/usr/share/elasticsearch/data
      - /home/ubuntu/es_logs:/usr/share/elasticsearch/logs
    ports:
      - "9200:9200"
      - "9300:9300"

  kibana:
    build:
      context: ../..
      dockerfile: docker/elasticsearch/Dockerfile-kibana
    container_name: kibana_prod
    depends_on:
      - elasticsearch
    environment:
      TZ: Asia/Seoul
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
      SERVER_BASEPATH: /kibana
      SERVER_REWRITEBASEPATH: true
    ports:
      - "5601:5601"
```

주요 설정:

- **Elasticsearch 설정**:
  - 시간대 설정: 한국 시간대(Asia/Seoul) 사용
  - 메모리 설정: 메모리 락 제한 해제
  - 볼륨 마운트: 데이터와 로그 디렉토리 영속성 제공
  - 포트 매핑: 9200(HTTP), 9300(노드 간 통신) 노출
- **Kibana 설정**:
  - Elasticsearch 의존성 설정
  - 시간대 설정: 한국 시간대 사용
  - 기본 경로 설정: `/kibana` 경로에서 서비스
  - 포트 매핑: 5601 포트 노출

## 5. Elasticsearch 최적화 및 운영 가이드

### 5.1 인덱스 설계 및 매핑 최적화

#### 5.1.1 인덱스 설계 전략

- **시간 기반 인덱스**: 로그 데이터는 날짜별로 인덱스 분리
- **샤드 크기 최적화**: 샤드당 10-50GB 유지
- **적절한 샤드 수**: 노드 수와 코어 수 고려 (일반적으로 노드당 1-3개)
- **인덱스 템플릿 활용**: 표준화된 매핑 및 설정 적용

```json
// 인덱스 템플릿 예시
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "refresh_interval": "5s",
      "index.lifecycle.name": "logs_policy"
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "message": { "type": "text" },
        "level": { "type": "keyword" },
        "service": { "type": "keyword" },
        "trace_id": { "type": "keyword" },
        "user_id": { "type": "keyword" }
      }
    }
  }
}
```

#### 5.1.2 매핑 최적화

- **적절한 필드 타입 선택**:
  - 정확한 검색: `keyword` 타입
  - 전문 검색: `text` 타입
  - 숫자 데이터: `integer`, `long`, `float`, `double`
  - 날짜/시간: `date`
  - 구조화된 객체: `object` 또는 `nested`
  - 지리적 데이터: `geo_point`, `geo_shape`
- **불필요한 필드 제외**: `_source` 필터링 활용
- **필드 압축**: `_source` 압축 활용
- **동적 매핑 제어**: 명시적 매핑 또는 동적 매핑 템플릿 사용

```json
// 매핑 최적화 예시
PUT products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index.mapping.coerce": false
  },
  "mappings": {
    "_source": {
      "enabled": true,
      "includes": ["id", "name", "price", "category", "created_at"]
    },
    "properties": {
      "id": { "type": "keyword" },
      "name": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword", "ignore_above": 256 }
        }
      },
      "description": { "type": "text" },
      "price": { "type": "scaled_float", "scaling_factor": 100 },
      "category": { "type": "keyword" },
      "tags": { "type": "keyword" },
      "created_at": { "type": "date" },
      "updated_at": { "type": "date" },
      "attributes": { "type": "object", "enabled": false }
    },
    "dynamic": "strict"
  }
}
```

### 5.2 분석기(Analyzer) 최적화

#### 5.2.1 한국어 분석기 (Nori) 설정

```json
// 한국어 분석기 설정 예시
PUT korean_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "korean": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": [
            "nori_posfilter",
            "lowercase",
            "synonym_filter",
            "nori_readingform"
          ]
        },
        "korean_mixed": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": [
            "nori_posfilter",
            "lowercase",
            "synonym_filter",
            "mixed_nori_filter"
          ]
        }
      },
      "tokenizer": {
        "nori_tokenizer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed",
          "discard_punctuation": "false",
          "user_dictionary": "analysis/userdict_ko.txt"
        }
      },
      "filter": {
        "nori_posfilter": {
          "type": "nori_part_of_speech",
          "stoptags": [
            "E", "IC", "J", "MAG", "MAJ", "SC", "SE", "SF", "SP", "SSC", "SSO", "SY", "VCP", "VCN", "VSV"
          ]
        },
        "synonym_filter": {
          "type": "synonym",
          "synonyms_path": "analysis/synonyms.txt"
        },
        "mixed_nori_filter": {
          "type": "nori_part_of_speech",
          "stoptags": [
            "E", "IC", "J", "MAJ", "SC", "SE", "SF", "SP", "SSC", "SSO", "SY", "VCP", "VCN"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "korean",
        "fields": {
          "mixed": {
            "type": "text",
            "analyzer": "korean_mixed"
          },
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "content": {
        "type": "text",
        "analyzer": "korean"
      }
    }
  }
}
```

#### 5.2.2 멀티필드 및 검색 최적화

```json
// 제품 검색을 위한 멀티필드 분석기 설정
PUT products
{
  "settings": {
    "analysis": {
      "analyzer": {
        "folding_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        },
        "edge_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "edge_ngram_filter"
          ]
        }
      },
      "filter": {
        "edge_ngram_filter": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 20
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "korean_mixed",
        "fields": {
          "standard": {
            "type": "text"
          },
          "folded": {
            "type": "text",
            "analyzer": "folding_analyzer"
          },
          "suggest": {
            "type": "text",
            "analyzer": "edge_ngram_analyzer",
            "search_analyzer": "standard"
          },
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 5.3 인덱스 라이프사이클 관리 (ILM)

#### 5.3.1 ILM 정책 설정

```json
// 로그 데이터를 위한 ILM 정책
PUT _ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_size": "20gb"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "2d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "7d",
        "actions": {
          "set_priority": {
            "priority": 0
          },
          "allocate": {
            "require": {
              "data": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

#### 5.3.2 롤오버 인덱스 설정

```json
// 롤오버 인덱스 초기 설정
PUT logs-000001
{
  "aliases": {
    "logs": {
      "is_write_index": true
    }
  }
}

// 롤오버 API 호출 (ILM으로 자동화 가능)
POST logs/_rollover
{
  "conditions": {
    "max_age": "1d",
    "max_docs": 100000,
    "max_size": "20gb"
  }
}
```

### 5.4 메모리 및 성능 최적화

#### 5.4.1 JVM 힙 메모리 설정

```yaml
# elasticsearch.yml 설정
ES_JAVA_OPTS: "-Xms4g -Xmx4g" # RAM의 50% 정도로 설정
```

주요 메모리 가이드라인:

- 최대 힙 크기는 물리 RAM의 50% 이하로 설정 (최대 32GB까지)
- 최소 힙과 최대 힙 크기를 동일하게 설정
- 힙 크기와 시스템 메모리 사이에 충분한 공간 확보 (파일 시스템 캐시용)

#### 5.4.2 샤드 및 필드 캐시 설정

```yaml
# elasticsearch.yml 설정
indices.fielddata.cache.size: 20% # 힙의 20%로 제한
indices.memory.index_buffer_size: 15% # 힙의 15%로 설정
```

#### 5.4.3 쿼리 성능 최적화

```json
// 검색 요청 최적화 예시
GET products/_search
{
  "size": 20,
  "track_total_hits": false,  // 결과 수가 많을 때 카운팅 제한
  "_source": ["id", "name", "price"],  // 필요한 필드만 가져오기
  "query": {
    "bool": {
      "filter": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "gte": 100, "lte": 500 } } }
      ],
      "should": [
        { "match": { "name": "smartphone" } }
      ]
    }
  }
}
```

쿼리 최적화 팁:

- 필터 사용: 점수 계산이 필요 없는 조건은 `filter` 절 사용
- `_source` 필터링: 필요한 필드만 로드
- 페이지네이션 제한: 딥 페이지네이션 대신 `search_after` 사용
- 배치 처리: 대량 인덱싱은 벌크 API 사용
- 스크립트 제한: 스크립트 사용 최소화 (성능 저하 유발)

## 6. Kibana 활용 및 대시보드 구성

### 6.1 기본 구성 요소

#### 6.1.1 Discover

데이터 탐색 도구로, 인덱스 패턴을 통해 데이터를 탐색하고 필터링합니다.

주요 기능:

- 실시간 검색 쿼리 작성
- 필드 선택 및 정렬
- 필터 생성 및 저장
- 시간 범위 설정
- 결과 내보내기

#### 6.1.2 시각화(Visualize)

데이터 시각화 도구로, 다양한 차트 및 그래프를 생성합니다.

주요 시각화 유형:

- 막대, 선, 영역 차트
- 파이, 도넛 차트
- 히트맵, 히트 지도
- 데이터 테이블
- 메트릭 (단일 값)
- 태그 클라우드
- TSVB (시계열 시각화)
- Vega 및 Vega-Lite (고급 시각화)

#### 6.1.3 대시보드(Dashboard)

시각화를 조합하여 통합 뷰를 제공하는 공간입니다.

주요 기능:

- 시각화 배치 및 크기 조정
- 필터 적용 및 공유
- 시간 범위 조정
- 자동 새로고침 설정
- PDF/PNG 내보내기

#### 6.1.4 캔버스(Canvas)

데이터 기반 프레젠테이션 도구로, 대화형 문서를 만듭니다.

#### 6.1.5 Lens

직관적인 드래그 앤 드롭 인터페이스를 제공하는 시각화 도구입니다.

### 6.2 대시보드 구성 사례

#### 6.2.1 로그 모니터링 대시보드

```json
// 저장된 대시보드 구성 예시
{
  "attributes": {
    "title": "애플리케이션 로그 대시보드",
    "hits": 0,
    "description": "애플리케이션 로그 모니터링을 위한 대시보드",
    "panelsJSON": "[{\"id\":\"logs-overview\",\"type\":\"visualization\",\"panelIndex\":\"1\",\"gridData\":{\"x\":0,\"y\":0,\"w\":24,\"h\":10},\"version\":\"7.10.0\"},{\"id\":\"error-logs\",\"type\":\"visualization\",\"panelIndex\":\"2\",\"gridData\":{\"x\":0,\"y\":10,\"w\":12,\"h\":10},\"version\":\"7.10.0\"},{\"id\":\"log-sources\",\"type\":\"visualization\",\"panelIndex\":\"3\",\"gridData\":{\"x\":12,\"y\":10,\"w\":12,\"h\":10},\"version\":\"7.10.0\"},{\"id\":\"log-table\",\"type\":\"search\",\"panelIndex\":\"4\",\"gridData\":{\"x\":0,\"y\":20,\"w\":24,\"h\":15},\"version\":\"7.10.0\"}]",
    "optionsJSON": "{\"hidePanelTitles\":false,\"useMargins\":true,\"syncColors\":false}",
    "version": 1,
    "timeRestore": true,
    "timeTo": "now",
    "timeFrom": "now-24h",
    "refreshInterval": {
      "pause": false,
      "value": 30000
    }
  }
}
```

#### 6.2.2 전자상거래 분석 대시보드

전자상거래 사이트의 판매 및 고객 행동 분석을 위한 대시보드:

주요 구성 요소:

- 총 매출 및 주문 메트릭
- 시간별 매출 추이 차트
- 카테고리별 매출 분포 파이 차트
- 상위 제품 데이터 테이블
- 지역별 판매 히트 지도
- 고객 유입 경로 차트
- 검색어 트렌드 시각화

#### 6.2.3 시스템 모니터링 대시보드

서버 및 애플리케이션 성능 모니터링을 위한 대시보드:

주요 구성 요소:

- CPU, 메모리, 디스크 사용량 게이지
- 시간별 리소스 사용량 그래프
- 네트워크 트래픽 시각화
- 응답 시간 및 오류율 차트
- 로그 수준별 카운트 히스토그램
- 상위 오류 메시지 데이터 테이블
- 서비스 상태 히트맵

### 6.3 Kibana 고급 기능

#### 6.3.1 Kibana Lens를 활용한 직관적 시각화

```json
// Lens 시각화 구성 예시
{
  "id": "product-sales-lens",
  "type": "lens",
  "attributes": {
    "title": "제품별 매출 분석",
    "state": {
      "visualization": {
        "layerId": "layer1",
        "layerType": "data",
        "seriesType": "bar_stacked",
        "xAccessor": "category",
        "yScaleType": "linear",
        "splitAccessor": "date_histogram"
      },
      "datasourceStates": {
        "indexpattern": {
          "layers": {
            "layer1": {
              "columns": {
                "col1": {
                  "dataType": "string",
                  "isBucketed": true,
                  "label": "카테고리",
                  "operationType": "terms",
                  "params": {
                    "field": "category.keyword",
                    "orderBy": "col2",
                    "orderDirection": "desc",
                    "size": 10
                  }
                },
                "col2": {
                  "dataType": "number",
                  "isBucketed": false,
                  "label": "총 매출",
                  "operationType": "sum",
                  "params": {
                    "field": "price"
                  }
                },
                "col3": {
                  "dataType": "date",
                  "isBucketed": true,
                  "label": "월별",
                  "operationType": "date_histogram",
                  "params": {
                    "field": "created_at",
                    "interval": "1M"
                  }
                }
              },
              "indexPatternId": "sales-*"
            }
          }
        }
      }
    }
  }
}
```

#### 6.3.2 업타임 모니터링 설정

```json
// Uptime API 설정 예시
POST _watcher/watch/service_health
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "http": {
      "request": {
        "scheme": "https",
        "host": "api.example.com",
        "port": 443,
        "method": "get",
        "path": "/health",
        "params": {},
        "headers": {},
        "auth": {
          "basic": {
            "username": "monitor",
            "password": "secret"
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.status_code": {
        "not_eq": 200
      }
    }
  },
  "actions": {
    "log_error": {
      "logging": {
        "level": "error",
        "text": "서비스 상태 확인 실패: {{ctx.payload.status_code}}"
      }
    },
    "notify_slack": {
      "slack": {
        "account": "monitoring",
        "message": {
          "from": "Elasticsearch 모니터링",
          "to": ["#alerts"],
          "text": "API 상태 확인 실패: {{ctx.payload.status_code}}",
          "attachments": {
            "title": "경보 상세 정보",
            "text": "서비스가 응답하지 않거나 오류를 반환했습니다.",
            "color": "danger"
          }
        }
      }
    }
  }
}
```

#### 6.3.3 알림 설정

```json
// Elasticsearch Alerting 설정 예시
POST _alerting/rules
{
  "name": "High CPU Usage Alert",
  "enabled": true,
  "schedule": {
    "interval": "1m"
  },
  "monitoring_indices": ["metrics-*"],
  "inputs": [{
    "search": {
      "indices": ["metrics-*"],
      "query": {
        "bool": {
          "filter": [
            { "range": { "@timestamp": { "gte": "now-2m", "lte": "now" } } },
            { "range": { "system.cpu.total.pct": { "gte": 0.8 } } }
          ]
        }
      }
    }
  }],
  "condition": {
    "script": {
      "source": "return ctx.results[0].hits.total.value > 0",
      "lang": "painless"
    }
  },
  "actions": [
    {
      "name": "email_admin",
      "destination_id": "email_destination",
      "message_template": {
        "source": "High CPU usage detected on {{data.hits.hits.0._source.host.name}}",
        "lang": "mustache"
      }
    }
  ]
}
```

## 7. Elasticsearch와 애플리케이션 통합

### 7.1 Spring Boot 통합

#### 7.1.1 Spring Data Elasticsearch 설정

```java
// Elasticsearch 설정
@Configuration
@EnableElasticsearchRepositories(basePackages = "com.example.repository")
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {

    @Value("${elasticsearch.host}")
    private String host;

    @Value("${elasticsearch.port}")
    private int port;

    @Override
    public RestHighLevelClient elasticsearchClient() {
        ClientConfiguration clientConfiguration = ClientConfiguration.builder()
            .connectedTo(host + ":" + port)
            .build();

        return RestClients.create(clientConfiguration).rest();
    }

    @Bean
    public ElasticsearchOperations elasticsearchOperations() {
        return new ElasticsearchRestTemplate(elasticsearchClient());
    }
}
```

#### 7.1.2 엔티티 및 리포지토리 설정

```java
// Elasticsearch 문서 엔티티
@Document(indexName = "products")
public class Product {

    @Id
    private String id;

    @Field(type = FieldType.Text, name = "name", analyzer = "korean")
    private String name;

    @Field(type = FieldType.Text, name = "description", analyzer = "korean")
    private String description;

    @Field(type = FieldType.Keyword, name = "category")
    private String category;

    @Field(type = FieldType.Double, name = "price")
    private Double price;

    @Field(type = FieldType.Nested, name = "attributes")
    private List<Attribute> attributes;

    @Field(type = FieldType.Date, name = "created_at")
    private Date createdAt;

    // getters and setters
}

// 중첩 문서 타입
@JsonInclude(JsonInclude.Include.NON_EMPTY)
public class Attribute {

    @Field(type = FieldType.Keyword)
    private String name;

    @Field(type = FieldType.Keyword)
    private String value;

    // getters and setters
}

// Elasticsearch 리포지토리
public interface ProductRepository extends ElasticsearchRepository<Product, String> {

    // 기본 CRUD 메소드 자동 제공

    // 커스텀 쿼리 메소드
    List<Product> findByCategory(String category);

    List<Product> findByPriceBetween(Double minPrice, Double maxPrice);

    @Query("{\"bool\": {\"must\": [{\"match\": {\"name\": \"?0\"}}]}}")
    Page<Product> findByNameCustomQuery(String name, Pageable pageable);

    @Query("{\"bool\": {\"must\": [{\"match\# Elasticsearch & Kibana 플랫폼 상세 문서

## 1. Elasticsearch 개요

Elasticsearch는 Lucene 기반의 분산형 RESTful 검색 및 분석 엔진으로, 대용량 데이터를 빠르게 저장, 검색, 분석할 수 있게 해주는 오픈 소스 도구입니다. 실시간 데이터 처리, 전문 검색(full-text search), 로그 분석, 메트릭 저장 등 다양한 사용 사례에 적합합니다.

### 1.1 핵심 특징

- **분산 아키텍처**: 수평적 확장성을 제공하는 분산 시스템
- **실시간 검색 및 분석**: 데이터 색인화 즉시 검색 가능
- **다양한 데이터 타입 지원**: 정형, 비정형, 지리적 데이터 등 지원
- **RESTful API**: HTTP를 통한 간단한 API 호출 방식
- **스키마리스**: 필드를 사전에 정의할 필요 없이 문서 추가 가능
- **고급 쿼리 언어**: 강력하고 유연한 JSON 기반 쿼리 언어
- **집계(Aggregations)**: 데이터 분석과 통계 기능 제공
- **멀티테넌시**: 단일 클러스터에서 여러 인덱스 격리 관리
- **플러그인 아키텍처**: 다양한 확장 기능 제공

### 1.2 주요 활용 사례

- **엔터프라이즈 검색**: 웹사이트, 앱, 문서 등의 검색 기능 구현
- **로그 및 이벤트 데이터 분석**: 시스템 로그, 애플리케이션 로그, 보안 이벤트 분석
- **메트릭 분석**: 시스템 및 비즈니스 메트릭 저장 및 분석
- **자연어 처리**: 텍스트 분석, 감정 분석, 문서 분류 등
- **보안 분석**: 이상 행동 탐지, 보안 이벤트 분석
- **비즈니스 인텔리전스**: 대시보드 및 보고서용 데이터 제공
- **지리공간 분석**: 위치 기반 검색 및 분석

## 2. Kibana 개요

Kibana는 Elasticsearch를 위한 오픈 소스 시각화 및 분석 플랫폼으로, Elasticsearch에 저장된 데이터를 탐색, 시각화, 대시보드화하는 도구입니다. Elastic Stack(이전의 ELK Stack)의 핵심 구성 요소 중 하나입니다.

### 2.1 핵심 특징

- **데이터 탐색**: Elasticsearch 인덱스의 데이터를 쉽게 탐색
- **시각화 도구**: 차트, 그래프, 맵 등 다양한 시각화 생성
- **대시보드**: 여러 시각화를 조합한 대시보드 구성
- **Canvas**: 프레젠테이션 품질의 동적 시각화 작성
- **Lens**: 드래그 앤 드롭 방식의 시각화 도구
- **Timelion**: 시계열 데이터 시각화 특화 기능
- **Dev Tools**: Elasticsearch API와 상호작용하는 개발자 도구
- **마크다운 지원**: 대시보드에 설명 추가 가능
- **공유 및 내보내기**: 대시보드와 시각화 공유 및 PDF 내보내기

### 2.2 주요 활용 사례

- **로그 모니터링**: 애플리케이션 로그 실시간 분석
- **인프라 모니터링**: 서버, 네트워크, 컨테이너 메트릭 관찰
- **비즈니스 인텔리전스**: 데이터 기반 의사결정 지원
- **보안 분석**: 보안 이벤트 시각화 및 이상 징후 탐지
- **지리공간 분석**: 위치 데이터 시각화
- **APM(Application Performance Monitoring)**: 애플리케이션 성능 모니터링

## 3. Elasticsearch 아키텍처 및 동작 원리

### 3.1 핵심 개념

#### 3.1.1 노드(Node)
Elasticsearch의 단일 서버 인스턴스로, 데이터를 저장하고 클러스터의 색인화 및 검색 기능에 참여합니다.

**노드 유형**:
- **마스터 노드(Master Node)**: 클러스터 상태 관리 및 조정
- **데이터 노드(Data Node)**: 데이터 저장 및 CRUD 작업, 검색, 집계 수행
- **인제스트 노드(Ingest Node)**: 인덱싱 전 데이터 전처리 수행
- **코디네이팅 노드(Coordinating Node)**: 클라이언트 요청 라우팅, 병합 등 담당
- **머신러닝 노드(ML Node)**: 머신러닝 작업 수행

#### 3.1.2 클러스터(Cluster)
하나 이상의 노드가 모여 이루는 집합으로, 모든 데이터를 공동으로 보유하고 모든 노드에서 통합 인덱싱 및 검색 기능을 제공합니다.

#### 3.1.3 인덱스(Index)
비슷한 특성을 가진 문서들의 모음으로, 관계형 데이터베이스의 테이블과 유사한 개념입니다. 인덱스는 역색인(inverted index)을 사용하여 빠른 검색을 가능하게 합니다.

#### 3.1.4 문서(Document)
정보를 저장하는 기본 단위로, JSON 형식으로 표현됩니다. 관계형 데이터베이스의 행(row)과 유사합니다.

#### 3.1.5 샤드(Shard)
인덱스를 여러 조각으로 나눈 것으로, 수평적 확장과 병렬 처리를 가능하게 합니다. 샤드는 자체적인 완전한 Lucene 인덱스입니다.
- **프라이머리 샤드(Primary Shard)**: 각 문서가 최초로 저장되는 샤드
- **레플리카 샤드(Replica Shard)**: 프라이머리 샤드의 복제본으로 고가용성 및 읽기 성능 향상

#### 3.1.6 매핑(Mapping)
문서와 그 필드가 저장되고 인덱싱되는 방법을 정의하는 스키마 정의입니다. 관계형 데이터베이스의 테이블 스키마와 유사합니다.

### 3.2 데이터 처리 흐름

#### 3.2.1 인덱싱(Indexing) 과정
1. **문서 제출**: 클라이언트가 `POST` 또는 `PUT` 요청으로 문서 제출
2. **라우팅**: 코디네이팅 노드가 문서의 ID를 기반으로 적절한 샤드로 라우팅
3. **저장**: 프라이머리 샤드가 문서를 저장
4. **복제**: 성공 시 레플리카 샤드에 복제
5. **응답**: 클라이언트에 확인 응답 반환

#### 3.2.2 검색(Search) 과정
1. **쿼리 제출**: 클라이언트가 `GET` 또는 `POST` 요청으로 검색 쿼리 제출
2. **쿼리 파싱**: 코디네이팅 노드가 쿼리 파싱 및 필요한 샤드 결정
3. **샤드 검색**: 각 관련 샤드(프라이머리 또는 레플리카)에 검색 요청 분배
4. **내부 검색**: 각 샤드가 Lucene을 사용하여 로컬 검색 수행
5. **결과 수집**: 코디네이팅 노드가 샤드별 결과 수집 및 병합
6. **결과 반환**: 통합된 결과를 클라이언트에 반환

#### 3.2.3 분석(Analysis) 과정
1. **토큰화(Tokenization)**: 텍스트를 개별 토큰(단어)으로 분리
2. **토큰 필터링**: 토큰 변환, 추가, 제거 등의 처리
3. **텀 생성**: 최종 인덱스 텀 생성
4. **역색인(Inverted Index) 구성**: 텀에서 문서로의 매핑 생성

### 3.3 역색인(Inverted Index) 구조

Elasticsearch의 핵심 검색 효율성은 역색인 구조에서 비롯됩니다:

1. **용어 사전(Term Dictionary)**: 모든 고유 텀(단어)의 정렬된 목록
2. **포스팅 목록(Postings List)**: 각 텀이 나타나는 문서 ID 목록
3. **텀 빈도(Term Frequency)**: 각 문서 내 텀 출현 빈도
4. **위치 정보(Position Information)**: 텀의 문서 내 위치 (구문 쿼리용)
5. **오프셋 정보(Offset Information)**: 텀의 시작/끝 위치 (하이라이팅용)

예시 역색인:
```

| Term   | Document IDs |
| ------ | ------------ |
| apple  | 1, 4, 7      |
| banana | 2, 5, 7      |
| cherry | 3, 5, 6      |

````

## 4. Elasticsearch와 Kibana 도커 구성 분석

현재 프로젝트에서 사용 중인 Elasticsearch와 Kibana 도커 구성을 살펴보겠습니다:

### 4.1 Elasticsearch Dockerfile 분석

```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch:8.13.4

# nori 플러그인 설치
RUN elasticsearch-plugin install analysis-nori

# --- 필요하다면 root 권한으로 전환 ---
USER root

# 디렉토리 권한 변경
RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/logs \
    && chmod -R 775 /usr/share/elasticsearch/logs

# --- 다시 elasticsearch 유저로 전환 ---
USER elasticsearch

# 사용자 사전, synonyms COPY
RUN mkdir -p /usr/share/elasticsearch/config/analysis
COPY ./docker/elasticsearch/userdict_ko.txt /usr/share/elasticsearch/config/analysis/userdict_ko.txt
COPY ./docker/elasticsearch/synonyms.txt /usr/share/elasticsearch/config/analysis/synonyms.txt

# 단일노드 모드, 보안 비활성
ENV discovery.type=single-node
ENV xpack.security.enabled=false
ENV ES_JAVA_OPTS "-Dfile.encoding=UTF-8 -Dclient.encoding.override=UTF-8"
````
