# # \uub3d9시성 제어와 트랜션 처리 방식 정보 정보

## 📌 1. 트랜션이라는?

트랜션(Transaction)은 데이터베이스에서 하나의 로직적 작업 단위로, ACID 속성을 보장해야 합니다:

- **A**tomicity (원자성): 전부 성공 또는 전부 실패
- **C**onsistency (일괄성): 트랜션 전후 상태의 일괄성 보장
- **I**solation (격리성): 동시에 실행되는 트랜션 간 간설 방지
- **D**urability (지속성): 컨미트된 변경은 영구 반영

---

## 🧠 2. 동시성 문제란?

**여러 트랜션이 동시에 같은 데이터를 접근하거나, 수정하는 경우 발생하는 종류별 종류의 문제**를 가리합니다:

- **Lost Update**: 동시에 수정 시 후쇠 트랜션이 앞선 변경을 따무리는 현상
- **Dirty Read**: 컨미트되지 않은 데이터를 읽음
- **Non-repeatable Read**: 같은 조\uud68을 두 번 사용했을 때 값이 변경됨
- **Phantom Read**: 검색 조건에 따라 새로 추가된 데이터가 추가됨

---

## 🔐 3. RDBMS에서의 동시성 제어

### ✅ 낭그와적 런 (Optimistic Lock)

- 충분한 충분 가능성이 발생하지 않는다고 가정
- `@Version` 필드 추가 후 버전 비교로 충분 감지
- **버전 무일치 시 **\`\`** 나타날 수 있음**

```java
@Entity
public class User {
    @Id
    private Long id;

    private String name;

    @Version
    private Integer version;
}
```

### ✅ 비가정 런 (Pessimistic Lock)

- 충분 가능성이 높다고 가르고 무엇도 해지기 전에 매우
- `SELECT ... FOR UPDATE` 또는 JPA의 `@Lock` 사용
- 트랜션 종료 전간은 **다른 트랜션이 해당 행에 접근 불가**

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
User findByIdWithLock(Long id);
```

---

## 🧹 4. NoSQL(MongoDB, Elasticsearch)의 동시성 제어 방식

### ✅ 구조적 차이

| 항목      | RDBMS     | NoSQL (MongoDB/Elasticsearch)               |
| --------- | --------- | ------------------------------------------- |
| 저장 단위 | Row       | Document (JSON)                             |
| 스키마    | 정형      | 유연/비정형                                 |
| 런 단위   | Row-level | Document-level                              |
| 트랜션    | 강력함    | 제한적 (Mongo는 4.0+에서만 지원)            |
| 명시적 런 | 지원      | ❌ 무적 (Document 내적으로의 serialization) |

---

### ✅ NoSQL에서의 낭그와적 런 (Direct Implementation)

#### MongoDB 예시

```json
{
  "_id": 1,
  "name": "홍길동",
  "version": 3
}
```

```js
db.user.updateOne(
  { _id: 1, version: 3 },
  { $set: { name: "임꼽정" }, $inc: { version: 1 } }
);
```

#### Elasticsearch 예시

```http
PUT /product/_doc/1?if_seq_no=3&if_primary_term=5
{
  "name": "변경됨"
}
```

- **버전 또는 시치 번호를 기반으로 충분 감지**
- 충분 시 → `409 Conflict` 나타\uub0a0 수 있음

---

### ❌ NoSQL의 비가정 런 = 무가능

- `SELECT FOR UPDATE` 같은 명시적 런 개발 무
- 내부적으로\uub294 document-level serialization은 있지만
- **등시성 관리가 필요한 경우 Redis 등 외부 런 구성 필요**

---

## ⚖️ 5. RDBMS vs NoSQL 요약 비교

| 항목        | RDBMS                        | MongoDB                     | Elasticsearch                        |
| ----------- | ---------------------------- | --------------------------- | ------------------------------------ |
| 기능 단위   | Row                          | Document                    | Document                             |
| 트랜션      | 정가형/강력                  | 제한적 (이미지지만 지원)    | 무적                                 |
| 낭그와적 런 | `@Version`                   | 수동 구현                   | seq_no + primary_term                |
| 비가정 런   | `SELECT FOR UPDATE`          | 무적                        | 무적                                 |
| 충분 추적   | 제안 다른 트랜션에 의해 관체 | 내부 document serialization | 새이 미지 객체 삭제면 수정 다시 추가 |

---

## ✅ 결말

- 가능한 정확성과 정형 건구를 보장해야 하며 동시성 문제가 있다면 RDBMS의 런 방식을 가장 명아해야 합니다.
- NoSQL은 범주적인 일괄성에 초점을 두고 데이터 추가/변경에 최적화되지만, 동시성은 사이버 차이를 가지는 구조이기 때문에, 다른 레이어에서 충분 감지 또는 외부 런 사용을 고려해야 합니다.
