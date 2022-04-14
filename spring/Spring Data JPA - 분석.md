# Spring Data JPA - 구현체 분석

# 1. 구현체 분석

- 실 구현체 : org.springframework.data.jpa.repository.support.SimpleJpaRepository
- Intelij에서 fx 누르면 이런 식으로 나온다!

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/image/Untitled.png)

- findByID의 예시
    - 우리가 아는 em.find 코드와 크게 다르지 않다!
    - 제네릭을 사용해서 구현되어 있다.

```java
@Override
	public Optional<T> findById(ID id) {

		Assert.notNull(id, ID_MUST_NOT_BE_NULL);

		Class<T> domainType = getDomainClass();

		if (metadata == null) {
			return Optional.ofNullable(em.find(domainType, id));
		}

		LockModeType type = metadata.getLockModeType();

		Map<String, Object> hints = new HashMap<>();
		getQueryHints().withFetchGraphs(em).forEach(hints::put);

		return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
	}
```

- @Repository 어노테이션
    - Persistent 계열의 Exception을 Spring 공통 Exception으로 변경
    - (당연히) Component Scan 받기 위해서도 사용
    - 이를 통해서 영속성 레이어 (JPA → JDBC 등..) 를 바꾸더라도 기존 Exception Handling 로직은 변경되지 않는다.
- @Transaction 어노테이션
    - 모든 함수는 Transaction 아래에서 동작한다.
    - 만약 Service Layer에서 이미 트랜잭션 시작됐다면, 해당 트랜잭션을 전파받아 사용된다.
    - 만약 Service Layer에서 트랜잭션이 없다면, Repository 계층에서 새 트랜잭션 만든다.
    - readOnly = true이면, flush를 생략해 약간의 성능 향상 일어난다

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
```

### Save() 메서드 (중요!)

- 새로운 Entity 면 저장 (Persist)
- 새로운 Entity 아니면 병합 (Merge)

- 리마인드 : 가능한 변경 감지를 쓰자..

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

	Assert.notNull(entity, "Entity must not be null.");

	if (entityInformation.isNew(entity)) {
		em.persist(entity); // 새로운 Entity면 저장
		return entity;
	} else {
		return em.merge(entity); // 아니면 Merge
	}
}
```

# 2. 새로운 Entity를 구별하는 방법

- Save() 호출시, 어떻게 새 Entity인지 알 수 있을까?

- 기본 전략
    - 식별자가 객체일 때 : null 이면 새로운 객체로 판단
    - 식별자가 Primitive type일 때 : 0이면 새 객체로 판단
    - **Persistable** Interface를 구현해 판단 로직 변경 가능

- @id, @GeneratedValue 지정시
    - id는 JPA에 Persist 할 때 생긴다!

- 문제가 되는 경우
    - @GeneratedValue를 사용하지 않고, 모종의 이유로 id를 직접 지정해주면?
        - 다른 팀이랑 ID 생성 전략을 통일해야 한다거나..
    
    ```java
    @Entity
    @Getter
    public class Item {
    
        @Id @GeneratedValue
        private Long id;
    
        protected Item() {
        }
    
        public Item(Long id) {
            this.id = id;
        }
    }
    
    // 이후 이런 코드라면?
    @Test
    public void save(){
    
        Item item = new Item(1l);
        itemRepository.save(item);
    }
    ```
    
    - id가 null이 아니므로, 새로운 객체로 인식하지 않는다!
        - 따라서 merge가 호출 (새 객체는 persist, 있는 객체는 merge)
            - Merge의 알고리즘
                1. DB에 값이 있다고 판단, DB에 Select 쿼리 날린다.
                2. Select 한 후, 값이 있으면 변경, 값이 없으면 Persist
        - 결국 Select문이 한번 더 나간다! 바로 Persist 하는 것 보다 비효율적이다!
    
    - 따라서, Persistable 인터페이스를 구현하면 해당 인터페이스를 이용해 판단할 수 있다.
        - 아래와 같이 CreatedDate를 이용하여 새로운 값인지를 확인할 수도 있다.
    
    ```java
    @Entity
    @EntityListeners(AuditingEntityListener.class)
    public class Item implements Persistable<String> {
    
        @Id @GeneratedValue
        private String id;
    
        @CreatedDate
        private LocalDateTime createdDate;
    
        protected Item() {
        }
    
        @Override
        public String getId() {
            return id;
        }
    
        // 이 Entity가 새 값인지 확인하는 로직을 구현
        @Override
        public boolean isNew() {
            // createdDate가 null이면 새로운 객체이다.
            return createdDate == null;
        }
    
        public Item(String id) {
            this.id = id;
        }
    }
    ```
