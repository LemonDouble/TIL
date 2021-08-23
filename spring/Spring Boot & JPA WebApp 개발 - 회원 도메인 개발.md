# Spring Boot & JPA WebApp 개발 - 회원 도메인 개발

분류: SPRING+JPA
작성일시: 2021년 8월 20일 오후 9:52

## 1. 개요

- 구현 기능
    - 회원 등록
    - 회원 목록 조회

## 2. 회원 리포지토리 개발

- jpabook.jpashop.repository/MemberRepository.class

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Member;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import java.util.List;

@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public void save(Member member){
        em.persist(member);
    }

    public Member findOne(Long id){
        return em.find(Member.class, id);
    }

    public List<Member> findAll(){
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name){
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name",name).getResultList();
    }

}
```

## 3. 회원 서비스 개발

- jpabook.jpashop.service/MemberService.class

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service

public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    /* 회원 가입 */

    //JPA 의 모든 데이터 변경, 로직은 트랜잭션 안에서 실행되어야 한다!
    @Transactional
    public Long join(Member member){
        validateDuplicateMember(member); //중복 회원 검증

        memberRepository.save(member);
        return member.getId();
    }

    @Transactional
    private void validateDuplicateMember(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());

        if(!findMembers.isEmpty()){
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    //회원 전체 조회
    //읽기 전용인 경우, 성능 최적화 등의 이점이 있다
    @Transactional(readOnly = true)
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    @Transactional(readOnly = true)
    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
}
```

## 4. 회원 테스트 작성

- 테스트 요구사항
    - 회원가입을 성공해야 한다.
    - 회원가입시 같은 이름이 있다면 예외가 발생해야 한다.

- test.jpabook.jpashop.service.MemberServiceTest.class

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(SpringExtension.class)
@SpringBootTest
//Transaction 걸고 Test, 테스트 후엔 rollback (Test Class에서만)
@Transactional
class MemberServiceTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;
    @Autowired EntityManager em;

    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long savedId = memberService.join(member);

        //then
        assertEquals(member, memberRepository.findOne(savedId));
    }

    @Test
    public void 중복_회원_예외() throws Exception {
        //IllegalStateException이 발생해야 한다!
        assertThrows(IllegalStateException.class, () -> {
            //given
            Member member1 = new Member();
            Member member2 = new Member();
            member1.setName("kim");
            member2.setName("kim");

            //when
            Long savedId1 = memberService.join(member1);
            Long savedId2 = memberService.join(member2); //예외가 발생해야 한다!

            //then
        });
    }

}
```