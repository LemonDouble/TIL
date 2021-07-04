# Spring : 테스트 케이스 작성

분류: Spring
작성일시: 2021년 7월 4일 오후 3:10

## 실행해서 테스트 할 때의 문제점

1. 준비하고 실행하는데 오래 걸림
2. 반복 실행하기 어렵다
3. 여러 테스트를 한번에 실행하기 어렵다

→ 자동화된 테스트 코드를 만들어 위와 같은 문제들을 해결

## Test Case 작성

- test/java/project명 밑에 테스트 작성

- 관례적으로 테스트할Class명Test 로 작성한다.

    (eg > main/java/project명/repository/MemoryMemberRepository의 Test Class는 

    test/java/project명/repository/MemoryMemberRepositoryTest 로 작성)

```java

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import java.util.List;

import static org.assertj.core.api.Assertions.*;

class MemoryMemberRepositoryTest {

    MemoryMemberRepository repository = new MemoryMemberRepository();

		// 각 Test case 후에 Memory에 지난 Test 값 남아있을 수 있으므로, 
		// Test 진행 후 상태를 초기화하는 코드가 필요하다.
    @AfterEach
    public void afterEach(){
        repository.clearStore();
    }

		//저장한 후의 값이 저장한 값과 같은가?
    @Test
    public void save(){
        Member member = new Member();
        member.setName("Spring");

        repository.save(member);

        Member result = repository.findById(member.getId()).get();

        assertThat(member).isEqualTo(result);
    }

		// 저장한 값을 FindbyName으로 불러왔을 때 같은 이름인가?
    @Test
    public void findByName(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        Member result = repository.findByName("spring1").get();

        assertThat(result).isEqualTo(member1);
    }

		//Memory에 있는 개수만큼 들어가 있는가?
    @Test
    public void findAll(){
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        List<Member> result = repository.findAll();

        assertThat(result.size()).isEqualTo(2);
    }
}
```

- 각 Test의 순서는 보장되지 않는다. 즉, 테스트는 순서 의존적으로 작성하면 안 된다.
- afterEach() 함수처럼, 각 Test 종료 후 메모리에 있는 값을 비워주는 함수가 반드시 필요하다.