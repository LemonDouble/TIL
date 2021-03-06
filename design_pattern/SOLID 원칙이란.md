# SOLID 원칙이란?

배운 내용 (한줄 요약): SRP, OCP, LSP, DIP, ISP
분류: 디자인 패턴
자료 링크: https://youtu.be/pJL4EuA6aGc , https://victorydntmd.tistory.com/291
작성일시: 2021년 6월 28일 오후 2:22
추가 : 2021년 7월 13일 오후 11:48


# SOLID : 시간이 지나도 유지 보수와 확장이 쉬운 시스템을 만들고자 할 때

→ OCP와 DIP는 꼭 기억하자!

### 1) Single responsibility principle : SRP (단일 책임의 원칙)

- 한 Class는 하나의 책임만을 가져야 한다. (* 책임 : 기능)
- 잘 설계된 프로그램은 새로운 요구사항과 프로그램 변경에 영향을 받는 부분이 적다.
- 즉, 응집도(구성 요소 간의 협응도) 는 높게, 결합도(구성 요소 간의 의존도) 는 낮게 설계한다.

- 하지만, 하나의 책임이란 것은 모호하다.
    - 하나의 책임이 클 수도, 작을 수도 있다.
    - 문맥/ 상황에 따라 다를 수 있다.

따라서, 중요한 기준으로 >>변경 << 이 있다.

변경이 있을 때 파급 효과가 적다면, 단일 책임 원칙을 잘 따른 것이다.

### 2) Open-Closed Principle : OCP (개방-폐쇄 원칙)

- 기존의 코드를 변경하지 않으면서(Closed), 기능을 추가할 수 있도록(Open) 설계가 되어야 한다. 즉, 확장에 대해서는 개방적, 수정에 대해서는 폐쇄적
- Interface와 상속을 사용, Interface를 상속하여 overriding 하면 수정에는 닫혀 있으면서(closed) 기능 추가에는 열려 있음(open)

```java
interface useDBinterface{
	public void useDB();
}

class useOracleDB implements useDBinterface{
	@Override
	public void useDB(){
		System.out.println("using orcale DB!");
	}
}

class useMariaDB implements useDBinterface{
	@Override
	public void useDB(){
		System.out.println("using maria DB!");
	}
}
```

- 다형성과 밀접하게 연관이 있다.
- 인터페이스를 구현한 새로운 클래스를 하나 만들어서, 새로운 기능을 구현하는 식으로 구현할 수 있다.

// 하지만 OCP의 문제점도 있다.

eg>

- MemberService가 다음과 같이 구현 클래스를 직접 선택한다고 하자.
    - MemberRepository m = new MemoryMemberRepository(); → 기존 CODE
    - MemberRepository m = new JdbcMemberRepository(); → 변경 CODE

이때, 구현 객체를 변경하기 위해 클라이언트 (MemberService) 의 변경이 필요하다.

즉, 다형성을 사용했지만 OCP 원칙을 지킬 수 없다.

→ 해결하기 위해선, 객체를 생성, 연관관계를 맺어주는 별도의 조립, 설정자가 필요하다.

### 3) Liskov Substitution Principle : LSP (리스코프 치환 원칙)

- 자식 클래스는, 부모 클래스에서 가능한 행위는 모두 수행 가능해야 한다. (즉, 자식 클래스는 언제나 부모 클래스를 대체할 수 있어야 한다.)
- 따라서 자식 클래스는 부모 클래스의 책임을 무시/재정의하지 않고 확장만을 수행해야 한다. 즉, 하위 클래스는 인터페이스 규약을 모두 정확히 지켜야 한다.
- LSP 원칙이 적용되면 상속, 적용되지 않으면 컴포지션을 사용하자.

eg > 자동차 인터페이스에서 엑셀은 앞으로 가야 한다. 만약 뒤로 가게 만든다면 LSP 위반, 느리게 가더라도 앞으로 가긴 해야 한다.

### 4) Interface Segregation Principle : ISP (인터페이스 분리 원칙)

- 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다. (인터페이스의 단일 책임 원칙)
- 하나의 거대한 인터페이스보다 여러 개의 구체적인 인터페이스가 낫다.
- 예를 들자면, Phone이라는 interface에 call(), sms(), alarm(), calculator()을 전부 구현하는 것보다, call, sms, alarm, calculator 인터페이스를 각 구현하여 4개의 인터페이스를 전부 상속받는게 낫다.

eg>

- 자동차 인터페이스가 있다면, 운전 인터페이스, 정비 인터페이스로 분리한다.
- 이렇게 함으로써 사용자 클라이언트 또한, 운전자 클라이언트, 정비사 클라이언트로 분리할 수 있다.

- 이렇게 분리한 경우, 정비 인터페이스 자체가 변해도 운전자 클라이언트에는 영향을 주지 않음.
- 즉, 인터페이스가 명확해지고 대체 가능성이 높아진다.

### 5) Dependency Inversion Principle : DIP (의존 역전 법칙)

- 의존 관계를 맺을 때, 변화하기 쉬운 것보다 변화하기 어려운 것에 의존해야 한다.
- 높은 수준의 개념 (인터페이스, 추상 클래스) 가 낮은 수준의 구현보다 더 안정적 (덜 변한다)
- high level ( 인터페이스나 추상 클래스 )에 인터페이스를 구현하고, 클라이언트는 가장 추상적인 클래스와 관계를 맺는다.

- 역할(ROLE)에 의존해야 한다는 것과 일맥상통한다.
- 객체 또한, 클라이언트가 인터페이스에 의존해야 유연하게 구현체를 변경할 수 있다.

eg>

- MemberService가 다음과 같이 구현 클래스를 직접 선택한다고 하자.
    - MemberRepository m = new MemoryMemberRepository(); → 기존 CODE
    - MemberRepository m = new JdbcMemberRepository(); → 변경 CODE

이 경우, OCP도 위반하였지만 MemberService는 구현 클래스에도 동시에 의존하므로, DIP 위반이기도 하다.