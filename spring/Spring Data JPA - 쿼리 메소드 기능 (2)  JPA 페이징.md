# Spring Data JPA - 쿼리 메소드 기능 (2) : JPA 페이징

# 1. 순수 JPA 페이징

- 예시
    - 검색 조건 : 나이가 10살
    - 정렬 조건 : 이름으로 내림차순
    - 페이징 조건 : 첫 페이지, 페이지당 보여줄 데이터는 3건

- 순수 JPA로는 다음과 같이 짤 수 있다.

```java
// Pagaing, age와 같은 나이, offset은 시작 지점,  limit은 몇 개?
public List<Member> findByPage(int age, int offset, int limit){
    // setFirstResult : 처음 시작 지점, setMaxResults : 페이징
    return em.createQuery("select m from Member m where m.age= :age order by m.username desc")
            .setParameter("age", age)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}

// 전체 개수! 해당 검색 조건 맞는 결과가 총 몇 개인지 필요하므로!
// 총 4321개중 3개 같은 식에서, 4321개가 totalCount
public long totalCount(int age){
    return em.createQuery("select count(m) from Member m where m.age = :age",Long.class)
            .setParameter("age", age)
            .getSingleResult();
}
```

# 2. Spring Data JPA 페이징과 정렬

- Interface 이용하여 페이징과 정렬을 표준화!!
    - org.springframework.data.domain.Sort : 정렬
    - org.springframework.data.domain.Pageable : 페이징 기능

- 특별한 반환 타입
    - [org.springframework.data.domain.Page](http://org.springframework.data.domain.Page) : 총 개수(totalCount) 쿼리 결과를 포함하는 페이징
    - org.springframework.data.domain.Slice : 총 개수 없는 페이징. (내부적으론 limit + 1 조회)
        - 모바일 등에 있는 더 보기 / 무한 스크롤 등!
    - List ( Java Collections ) : 추가 Count 쿼리 없이 결과만 반환!

- Repository 구현

```java
Page<Member> findByAge(int age, Pageable pageable);
Slice<Member> findSliceByAge(int age, Pageable pageable);
```

- 사용

```java
@Test
    public void paging_by_page(){
        //given
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        int age = 10;
        // Spring Data JPA에서는 Page가 0부터 시작!
        // 페이징 기능에서 정렬도 제공한다
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

        //when
        Page<Member> page = memberRepository.findByAge(age, pageRequest);
        List<Member> content = page.getContent();
        long totalElements = page.getTotalElements();
        //then
        for (Member member : content) {
            System.out.println("member = " + member);
        }

        assertEquals(3, content.size());
        assertEquals(5, totalElements);
        // 현재 페이지 번호도 알려준다!
        assertEquals(0, page.getNumber());
        // 총 페이지 개수도 알려준다!
        assertEquals(2, page.getTotalPages());
        // 첫 페이지인지 알려준다!
        assertTrue(page.isFirst());
        // 다음 페이지가 있는지도 알려준다!
        assertTrue(page.hasNext());
    }
```

- Total Count의 경우, 최적화의 여지가 있다.
    - 예를 들어 Join 해서 Content 구했는데, N:1 관계에서 N으로 Outer Join 해서 총 Column 개수가 같다면?
        - 예를 들어서 Member를 기준으로 Team을 Outer Join 해서 Member 개수와, Join 이후 Column 개수가 같다면?
        - Count Query는 단순히 Member Table만 봐도 충분할 수 있다.

- 다음과 같이 countQuery를 최적화 할 수 있다! (countQuery를 별도 지정)

```java
@Query(value="select m from Member m left join m.team t",
countQuery = "select count(m) from Member m")
Page<Member> findOptimizeQueryByAge(int age, Pageable pageable);
```

- Sorting 조건이 복잡해지면, 위와 같이 @Query 쓸 수 있다.
- 이후 리턴할땐 꼭 엔티티를 그대로 주지 말고, DTO로 주자!