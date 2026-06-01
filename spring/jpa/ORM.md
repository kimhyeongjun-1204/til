## 📌 ORM이란? — 객체-관계 매핑

### 🤔 ORM이 뭐야?

**ORM (Object-Relational Mapping)** = **객체-관계 매핑**.

> 자바 객체와 관계형 데이터베이스의 테이블을 **자동으로 매핑**해주는 기술

&nbsp;

---

### 🚨 ORM이 등장한 이유

자바는 **객체지향**, DB는 **관계형 테이블** 이라는 서로 다른 패러다임을 가져서 발생하는 **패러다임 불일치(Object-Relational Impedance Mismatch)** 문제를 해결하기 위해 등장했다.

> 참고: ORM은 Spring 전용 기술이 아니라 **자바 진영 전체에서 사용하는 개념**이다. Python의 SQLAlchemy, Ruby의 ActiveRecord 등 다른 언어에도 있다. 다만 Spring과 잘 어울려서 함께 자주 쓰일 뿐.

&nbsp;

---

### ⚙️ 패러다임 불일치의 4가지 문제

#### 문제 1. 상속

자바에는 상속이 있지만, RDB에는 상속 개념이 없다.

```java
class Item { ... }
class Album extends Item { ... }
class Book extends Item { ... }
```

```sql
CREATE TABLE item ( ... );
CREATE TABLE album ( ... );
CREATE TABLE book ( ... );
```

→ 자바의 상속 관계를 테이블에서 직접 표현 불가능.

&nbsp;

#### 문제 2. 연관 관계

자바는 객체 **참조(reference)**, DB는 **외래키(FK) + JOIN** 으로 연결한다.

```java
// 자바 — 객체 참조
class Order {
    private Member member;  // 객체 자체를 가짐
}
```

```sql
-- DB — 외래키
CREATE TABLE orders (
    member_id BIGINT  -- ID만 가짐
);
```

→ 연결 방식 자체가 다르다.

&nbsp;

#### 문제 3. 객체 그래프 탐색

자바에서는 자유롭게 객체를 타고 들어갈 수 있다.

```java
order.getMember().getTeam().getLeader();  // 무한정 가능
```

반면 DB는 **SQL문에 JOIN으로 미리 가져온 데이터만 탐색 가능**.

```sql
SELECT * FROM orders o
JOIN member m ON o.member_id = m.id;
-- Team을 JOIN 안 했으면 team 정보는 결과에 없음
```

&nbsp;

#### 문제 4. 동일성 비교

```java
// 자바
Member m1 = findMember(1L);
Member m2 = findMember(1L);
m1 == m2;  // false (참조가 다른 객체)

// DB
SELECT * FROM member WHERE id = 1;  // 같은 row
```

같은 데이터를 가리키는데 자바에서는 **다른 객체로 인식**.

&nbsp;

---

### 🚨 ORM이 없을 때 — JDBC로 수동 매핑

ORM 없이는 SQL 결과(`ResultSet`)에서 **컬럼 값을 하나씩 꺼내 set 메소드로 일일이 채워야** 한다.

```java
Member member = new Member();
member.setId(rs.getLong("id"));
member.setName(rs.getString("name"));
member.setEmail(rs.getString("email"));
// 컬럼이 30개면 30줄...
```

→ 반복 코드 폭발, 유지보수 지옥.

&nbsp;

---

### ✅ ORM이 하는 일

ORM을 쓰면 자바 메소드 호출 → 자동 SQL 변환 → DB 통신을 알아서 처리해준다.

| 작업 | ORM의 동작 |
|------|-----------|
| **객체 → DB 저장** | INSERT SQL 자동 생성 |
| **DB → 객체 조회** | SELECT SQL 자동 생성 + 결과를 객체로 자동 매핑 |
| **객체 수정** | **변경 감지(Dirty Checking)** → UPDATE 자동 생성 |
| **객체 삭제** | DELETE SQL 자동 생성 |
| **연관관계** | 객체 참조 → JOIN/FK로 자동 변환 |
| **DB 방언 처리** | MySQL, Oracle 등 DB별 SQL 차이 자동 처리 |

&nbsp;

---

### 📌 ORM ≠ JPA ≠ Hibernate

이 셋의 관계를 헷갈리면 안 된다.

```
[ ORM ]              ← 기술/개념 (객체-관계 매핑)
   └─ [ JPA ]            ← 자바의 ORM 표준 명세 (인터페이스)
         └─ Hibernate        ← JPA의 대표적인 구현체
```

| 용어 | 의미 |
|------|------|
| **ORM** | 객체-관계 매핑이라는 **개념** |
| **JPA** | 자바 진영의 ORM **표준 명세** |
| **Hibernate** | JPA를 **구현한 라이브러리** |

&nbsp;

---

### 🚨 ORM의 단점

ORM이 만능은 아니다. 다음과 같은 단점이 있다.

| 단점 | 설명 |
|------|------|
| **학습 곡선 가파름** | 영속성 컨텍스트, N+1, 변경 감지 등 알아야 할 개념 많음 |
| **복잡한 쿼리 어려움** | 통계·집계 같은 집합 연산은 ORM으로 표현 한계 |
| **성능 이슈** | N+1, 지연 로딩 오용 시 쿼리 폭발 |
| **자동 SQL의 한계** | ORM이 만든 SQL이 비효율적일 수 있음 (튜닝 필요) |

→ 그래서 실무에서는 **ORM + QueryDSL + JdbcTemplate** 등을 상황에 맞게 혼용한다.

&nbsp;

---

### 📌 정리

> ORM은 자바와 관계형 데이터베이스의 **패러다임 불일치를 완화**해주는 기술이다. SQL을 직접 작성하지 않아도 메소드 호출만으로 SQL이 자동 변환되어 DB와 통신할 수 있다.
>
> 하지만 **영속성 컨텍스트, N+1, 지연 로딩 오용** 같은 문제가 있어, 무조건 ORM만 쓰는 것이 아니라 **상황에 맞게 사용하는 것이 중요**하다.

&nbsp;


