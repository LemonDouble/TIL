# Spring : 회원 관리 예제 - 백엔드 개발

분류: Spring
작성일시: 2021년 6월 30일 오후 4:19

## 1. 비즈니스 요구사항 정리

- 데이터 : 회원 ID, 이름
- 기능 : 회원 등록, 조회
- 아직 데이터 저장소가 선정되지 않음 (가상 시나리오)

![Spring%20%E1%84%92%E1%85%AC%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%8B%E1%85%A8%E1%84%8C%E1%85%A6%20-%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%83%E1%85%B3%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%2077f32d9d001e46f2b528287bfaddf40a/Untitled.png](Spring%20%E1%84%92%E1%85%AC%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%8B%E1%85%A8%E1%84%8C%E1%85%A6%20-%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%83%E1%85%B3%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%2077f32d9d001e46f2b528287bfaddf40a/Untitled.png)

- 컨트롤러 : 웹 MVC의 컨트롤러 역할
- 서비스 : 핵심 비즈니스 로직 구현 (eg > 회원은 중복가입이 안 된다, 등)
- 리포지토리 : DB 접근, 도메인 객체를 DB에 저장/관리
- 도메인 : 비즈니스 도메인 객체 ( eg > 회원, 주문, 쿠폰 등. 주로 DB에 저장/관리됨)

![Spring%20%E1%84%92%E1%85%AC%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%8B%E1%85%A8%E1%84%8C%E1%85%A6%20-%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%83%E1%85%B3%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%2077f32d9d001e46f2b528287bfaddf40a/Untitled%201.png](Spring%20%E1%84%92%E1%85%AC%E1%84%8B%E1%85%AF%E1%86%AB%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%8B%E1%85%A8%E1%84%8C%E1%85%A6%20-%20%E1%84%87%E1%85%A2%E1%86%A8%E1%84%8B%E1%85%A6%E1%86%AB%E1%84%83%E1%85%B3%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%2077f32d9d001e46f2b528287bfaddf40a/Untitled%201.png)

- 데이터 저장소가 선정되지 않았으므로, 우선 인터페이스로 구현 클래스를 변경할 수 있도록 설계 (MemoryMemberRepository)
- DB 저장소는 RDB, NoSQL 등등 다양한 저장소를 고민중인 상황으로 가정
- 개발을 진행하기 위해서 초기 개발 단계에서는 구현체로 가벼운 메모리 기반 데이터 저장소 사용

## 2. 회원 도메인과 리포지토리 만들기

- main/java/project명/domain/Member (Class)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.domain;

public class Member {

    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- main/java/project명/repository/MemberRepository (Interface)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.repository;

import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;

import java.util.List;
import java.util.Optional;

public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```

- main/java/project명/repository/MemoryMemberRepository (class)

 

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.repository;

import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;

import java.util.*;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<Long, Member>();
    //실무에서는 Concurrent 문제 때문에 ConcurrentHashmap 사용해야 함

    private static long sequence = 0L;
    //이것도 마찬가지로 실무에서는 동시성 고려해야함

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name)).findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

    public void clearStore(){
        store.clear();
    }
}
```

→ 간단한 예제라 Concurrent, Race condition 등 고려하지 않았지만, 실무에서는 고려해야 함

## 3. 회원 서비스 개발

- Service 클래스는 좀 더 비즈니스와 비슷한 클래스명을 쓴다 (Join, findMembers..)
- 문제가 생겼을 때 쉽게 다른 분야의 사람들과 협업하기 위해!

- Repository → 개발 의존적 / Service → 비즈니스 의존적

- main/java/project명/service/MemberService (class)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.service;

import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;
import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemberRepository;
import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    /**
     * 회원 가입
     */
    public Long join(Member member){
        //같은 이름이 있는 중복 회원 X
        //Nullable한 값은 Optional로 반환했으므로, Optional에 포함된 isPresent 등의 메소드를 사용할 수 있다.
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
            .ifPresent(m-> {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });
    }

    /*
     * 전체 회원 조회
     */
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }

}
```

## 4. 회원 서비스 테스트

- Main의 클래스에서 Ctrl + Shift + T 로 자동으로 테스트를 만들 수 있다.
- Assertions.assertThat에서 alt + enter 통해서 Static import 하면 assertThat.~~로 쓸 수 있음
- Test는 한글로 작성해도 괜찮음! (직관성 위해서)
- Given (~가 주어졌을 때), When ( ~~를 실행했을 때) , then ( ~~가 나와야 한다)

- main/java/project명/service/MemberService (class) 를 수정

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.service;

import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;
import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemberRepository;
import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {

    private final MemberRepository memberRepository;

    //Dependency injection!!
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    /**
     * 회원 가입
     */
    public Long join(Member member){
        //같은 이름이 있는 중복 회원 X
        //Nullable한 값은 Optional로 반환했으므로, Optional에 포함된 isPresent 등의 메소드를 사용할 수 있다.
        validateDuplicateMember(member);

        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
            .ifPresent(m-> {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });
    }

    /*
     * 전체 회원 조회
     */
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }

}
```

- test/java/project명/service/MemberServiceTest (class)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.service;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;
import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemoryMemberRepository;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach(){
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach(){
        memberRepository.clearStore();
    }

    @Test
    void 회원가입() {
        //given
        Member member = new Member();
        member.setName("hello");

        //when
        Long saveId = memberService.join(member);

        //then
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    void 중복_회원_예외() {
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        //when
        memberService.join(member1);

        //우항(memberService.join()) 실행할 때, 좌항 (IllegalStateException.class) 가 터져야 한다.
        IllegalStateException e = assertThrows(IllegalStateException.class , () -> memberService.join(member2));
        //에러 메세지 같은지 확인
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

        /* try-catch 사용해서 작성해도 된다.
        try{
            memberService.join(member2);
            fail();
        }catch(IllegalStateException e) {
            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        }
         */
        //then
    }

    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```