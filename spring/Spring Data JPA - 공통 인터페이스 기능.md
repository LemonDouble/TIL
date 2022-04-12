# 1. 순수 JPA 기반 리포지토리

- 순수하게 JPA 기술만 사용해서 만든 리포지토리!

- MemberJpaRepository

```java
```

- TeamJpaRepository

```java
```

- 뭔가 비슷한 코드가 반복되는 것을 볼 수 있다.
- 제네릭? Interface? 뭔가 단순화 할 수 있지 않을까?

---

### Test Code

- MemberJpaRepsitoryTest

```java
```

# 2. 공통 인터페이스 설정

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

- 구현체가 없는데 어떻게 동작할까?
- 테스트 해 보자

```java
@Test
    public void JPARepository_정체확인(){
        System.out.println("memberRepository.getClass() = " + memberRepository.getClass());
    }

// 결과 : memberRepository.getClass() = class com.sun.proxy.$Proxy116
```

- Spring Data JPA가 컴포넌트 스캔 시, JPARepository 상속받은 클래스 있으면 구현 클래스를 생성
    - 이후 Proxy로 Insert 해 준다.

# 3. 공통 인터페이스 적용

# 4. 공통 인터페이스 분석

- Repository ← CrudRepository ← PagingAndSortRepository ← JpaRepository
- 밑으로 내려갈수록 해당 기술(JPA 등)에 특화된 기능을 제공!