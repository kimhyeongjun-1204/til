# [스프링 핵심 원리 - 기본편] AppConfig 리팩터링 & IoC / DI / 컨테이너

> 📅 2026.06.21 TIL · 김영한 스프링 핵심 원리 기본편

---

## ⚙️ 1. AppConfig 리팩터링 — 역할과 구현 분리

- 사용하는 곳에서 직접 `new` 하지 않고, **메서드를 통해 구현체를 생성**하도록 변경
- AppConfig만 봐도 **역할(인터페이스 = 반환 타입)** 과 **구현(생성하는 구현체)** 이 한눈에 보임

```java
@Configuration
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

&nbsp;

---

## ✅ 2. 정액 할인 → 정률 할인 변경

- 바꿔야 하는 부분? → **구성(설정)을 담당하는 `AppConfig` 단 한 곳**

```java
public DiscountPolicy discountPolicy() {
//  return new FixDiscountPolicy();   // 정액 할인
    return new RateDiscountPolicy();  // 정률 할인
}
```

- **사용 영역(`OrderServiceImpl` 등)의 코드는 단 한 줄도 변경 불필요**
- 사용 영역은 인터페이스(`DiscountPolicy`)에만 의존하기 때문 → **OCP / DIP 만족**

&nbsp;

---

## 📌 3. IoC, DI, 컨테이너

### 🤔 IoC (제어의 역전)
- 객체 생성·호출 등의 **제어권을 개발자가 아닌 프레임워크(외부)가 가져가는 것**
- 기존: 구현 객체가 스스로 필요한 객체를 생성·연결
- IoC: AppConfig 같은 외부가 제어 흐름을 담당

### 🤔 DI (의존관계 주입)
- `OrderServiceImpl`은 `MemberRepository`, `DiscountPolicy`에 의존
- 클래스 의존관계만으로는 **실제 어떤 객체가 주입될지 알 수 없음**
- DI를 사용하면 **정적인 클래스 의존관계는 그대로 두고**, **동적인 객체 인스턴스 의존관계를 쉽게 변경** 가능

| 구분 | 정적인 클래스 의존관계 | 동적인 객체 인스턴스 의존관계 |
|------|----------------------|------------------------------|
| 분석 시점 | 실행 없이 import만으로 분석 가능 | 런타임(실행 시점)에 결정 |
| 알 수 있는 것 | 어떤 인터페이스에 의존하는지 | 실제 어떤 구현 객체가 주입되는지 |
| 변경 시 | 코드 수정 필요 | 코드 수정 없이 변경 가능 |

### ⚙️ IoC 컨테이너 / DI 컨테이너
- AppConfig처럼 **객체를 생성·관리하고 의존관계를 연결**해주는 것
- 의존관계 주입에 초점 → 주로 **DI 컨테이너**라고 부름

&nbsp;

---

> 💡 **핵심 요약**
> 정적 의존관계(인터페이스)는 그대로 둔 채 동적 의존관계(구현 객체)만 바꾸는 것이 **DI**,
> 그 일을 대신 해주는 것이 **DI 컨테이너**(= AppConfig의 역할).