# 자바 ORM 표준 JPA 프로그래밍 - 엔티티 매핑

분류:  JPA
작성일시: 2021년 7월 24일 오후 3:33

- 엔티티 매핑 종류
    - 객체와 테이블 매핑 : @Entity, @Table
    - 필드와 컬럼 매핑 : @Column
    - 기본 키 매핑 : @Id
    - 연관관계 매핑 : @MnayToOne, @JoinColumn

## 1. 객체와 테이블 매핑

- @Entity
    - @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
    - JPA를 사용해 테이블과 매핑할 클래스는 @Entity가 필수
    - 주의점
        - 기본 생성자 필수 (파라미터 없는 public, protected 생성자)
        - final 클래스, enum, interface, inner 클래스 사용 불가
        - 저장할 필드에 fianl 사용 X

    - 속성 : name
        - JPA에서 사용할 엔티티 이름을 지정한다.
        - 기본값 : 클래스 이름을 그대로 사용 (예: Member)
        - 같은 클래스 이름이 없다면, 가능한 기본값을 사용한다.

        ```java
        ```

- @Table
    - Table은 엔티티와 매핑할 테이블 지정
    - 속성
        - name : 매핑할 테이블 이름 (기본값 : 엔티티 이름을 사용)
        - catalog : 데이터베이스 catalog 매핑
        - schema : 데이터베이스 schema 매핑
        - uniqueConstraints(DDL) : DDL 생성 시 유니크 제약 조건 생성

        ```java
        ```

- 데이터베이스 스키마 자동 생성
    - DDL (Database Definition Language : Create, Alter, Drop, Truncate) 를 어플리케이션 생성 시점에 자동 생성
    - 테이블 중심 → 객체 중심
    - 객체 중심적으로 개발해도 Table을 생성할 수 있다
    - 데이터베이스 방언을 활용해 데이터베이스에 맞는 적절한 DDL 생성
    - 이렇게 생성된 DDL은 개발 장비에서만 사용
    - 생성된 DDL은 운영서버에서는 사용되지 않거나, 적절히 다듬은 후 사용

    - 속성 (META-INF/hibernate.hbm2ddl.auto)
        - crate : 기존 테이블 삭제 후 다시 생성 (DROP + CREATE)
        - create-drop : create와 같으나, 종료시점에 테이블 DROP
        - update : 변경분만 반영(운영DB에서는 사용하면 안 됨)
            - (ALTER 사용, ADD는 되지만 DROP은 안 됨 - DROP되면 큰일남!!)
        - validate : 엔티티와 테이블이 정상 매핑되었는지만 확인
            - (엔티티에 있는 Field들이 Table에 있는지 확인하는데 쓸 수 있다)
        - none : 사용하지 않음

- 주의점

    ### 운영 장비에는 절대 create, create-drop, update 사용하면 안 된다!

    - 개발 초기 단꼐에는 create, 또는 update
    - 테스트 서버에는 update 또는 validate
    - 스테이징과 운영 서버는 validate 또는 none

- DDL 생성 기능
    - 제약조건 추가: 회원 이름은 필수, 10자 초과 X
        - @Column(nullable = false, length = 10)
    - 유니크 제약조건 추가
        - @Table(uniqueConstraints ={@UniqueConstraint(name="NAME_AGE_UNIQUE", columnNames = {"NAME","AGE"})})
    - DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고, JPA의 실행 로직에는 옇양을 주지 않는다.

## 2. 필드와 컬럼 매핑

- 예시) 요구사항 추가
    1. 회원은 일반 회원과 관리자로 구분해야 한다.
    2. 회원 가입일과 수정일이 있어야 한다.
    3. 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java
}
```

- 매핑 어노테이션 정리
    - @Column : 컬럼 매핑
    - @Temporal : 날짜 타입 매핑
    - @Enumerated : enum 타입 매핑
    - @Lob : BLOB, CLOB 매핑
    - @Transient : 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)
        - 만약 DB에는 저장하지 않고, 메모리에서만 사용할 임시 변수가 필요하다면 사용

- @Column의 옵션
    - name : 필드와 매핑할 테이블의 컬럼 이름 (기본값 : 객체의 필드 이름)
    - insertable, updatable : 등록, 변경 가능 여부 (기본값 : TRUE)
    - nullable(DDL) : null 값의 허용 여부 설정. false로 설정하면 DDL 생성 시 NOT NULL 붙는다
    - unique(DDL) : @Table의 uniqueConstraints와 같지만, 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다. - (Unique 제약 조건의 이름을 설정하기 힘드므로 잘 사용하진 않는다.)
    - columnDefinition(DDL) : 데이터베이스 컬럼 정보를 직접 줄 수 있다. (eg> columnDefinition = "varchar(100) default 'EMPTY'"), (기본값 : 추론)
    - length(DDL) : 문자 길이 제약요건, String 타입에만 사용한다. (eg> length = 10..) (기본값 : 255)
    - precision, scale (DDL) : BigDecimal 타입에서 사용한다. (BigInteger도 사용할 수 있다.) precision은 소수점을 포함한 전체 자릿수를, scale은 소수의 자리수다. 아주 큰 숫자나, 정밀한 소수를 다루어야 할 때 사용한다. (기본값 : precision=19, scale=2)

- @Enumerated
    - 자바 Enum 타입을 매핑할 때 사용
    - 주의! ORDINAL 사용 X (기본 : ORDINAL)
        - ORDINAL은 숫자(0,1,2,3..) 으로 들어가는데, 만약 Enum 타입 늘어나면 문제 생길 수 있다! ( 예를 들어 기존 B, C를 DB에 0, 1로 저장했는데, 요구사항 늘어 A, B, C 되면 A =0, B = 1, C = 2로 데이터 꼬일 수 있다)

    - 옵션
        - value (기본 : ORIDNAL)
            - EnumType.ORDINAL : enum 순서를 데이터베이스에 저장
            - EnumType.STRING : enum 이름을 데이터베이스에 저장

- @Temporal
    - 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
    - 참고 : 만약 LocalDate, LocalDateTime을 사용할 때는 생략 가능 (최신 하이버네이트 지원)

    - 옵션
        - value
            - [TemporalType.DATE](http://temporaltype.DATE) : 날짜, 데이터베이스 DATE 타입과 매핑 (2013-10-11)
            - TemporalType.TIME : 시간, 데이터베이스 TIME 타입과 매핑 (11:11:11)
            - TemporalType.TIMESTAMP : 날짜와 시간, 데이터베이스 TIMESTAMP 타입과 매핑 (2013-10-11 11:11:11)

- @Lob
    - 데이터베이스의 BLOB, CLOB 타입과 매핑
    - @Lob에는 지정할 수 있는 속성이 없다.
    - 매핑하는 필드 타입이 문자면 CLOB, 나머지는 BLOB 매핑
        - CLOB : String, Char[], java.sql.CLOB
        - BLOB : byte[], java.sql.BLOB

- @Transient
    - 필드 매핑 X
    - 데이터베이스 저장, 조회 X
    - 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

## 3. 기본 키 매핑

- 기본 키 매핑 어노테이션
    - @Id
    - @GeneratedValue

    ```java
    ```

- 기본 키 매핑 방법
    - 직접 할당 : @Id만 사용
    - 자동 생성(@GeneratedValue)
        - IDENTITY : 데이터베이스에 위임, MYSQL
            - 기본 키 생성을 DB에 위임
            - 주로 MySQL, PostgreSQL, SQL Server, DB2에서 활용
            (예: MySQL의 AUTO_INCREMENT)
            - JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
            - AUTO_INCREMENT는 DB에 INSERT SQL을 실행한 후에 ID 값을 알 수 있음
            - IDENTITY 전략은, em.persist() 시점에 즉시 INSERT SQL을 실행하고, DB에서 식별자를 조회

            ```java
            ```

        - SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용, ORACLE
            - @SequenceGenerator 필요
                - 옵션
                    - name : 식별자 생성기 이름 (필수)
                    - sequenceName : DB에 등록된 시퀀스 이름
                    기본값 : hibernate_sequence
                    - initialValue : DDL 생성 시에만 사용, 시퀀스 DDL 생성할 때 처음 시작하는 수를 지정한다.
                    기본값 : 1
                    - allocationSize : 시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용됨, 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면, 이 값을 반드시 1로 설정해야 한다.
                    기본값 : 50
                    - catalog, schema : 데이터베이스 catalog, schema 이름
            - DB Sequence는 유일한 값을 순서대로 생성하는 특별한 DB 오브젝트
            - Oracle, PostgreSQL, DB2, H2 DB에서 사용

            ```java
            ```

        - TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용 가능
            - @TableGenerator 필요
                - 옵션
                    - name: 식별자 생성기 이름(필수)
                    - table : 키 생성 테이블 명
                    기본값 : hibernate_sequences
                    - pkColumnName : 시퀀스 컬럼 명
                    기본값: sequence_name
                    - valueColumnNa : 시퀀스 값 컬럼명
                    기본값 : next_val
                    - pkColumnValue : 키로 사용할 값 이름
                    기본값 : 엔티티 이름
                    - initalValue : 초기 값, 마지막으로 생성된 값이 기
                    기본값 : 0
                    - allocationSzie : 시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 사용)
                    기본값 : 50
                    - catalog, schema : 데이터베이스 catalog, schema 이름
                    - uniqueConstraints(DDL) : 유니크 제약 조건 지정 가능
            - 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
                - 장점 : 모든 DB에 적용 가능
                - 단점 : 성능

            ```java
            ```

        - AUTO : (기본값) 방언에 따라 자동 지정

- 권장하는 식별자(PK) 전략
    - 기본 키 제약 조건: NOT NULL, UNIQUE, 변하면 안 된다.
    - 미래까지 이 조건을 만족하는 자연키를 찾기 어렵다. 대리 키 (대체 키) 를 사용하자.
        - 자연 키 : (비즈니스적으로 의미있는 key, 전화번호, 주민번호 등..)
        - 대체 키 : 비즈니스적으로 전혀 의미없는 key (UUID를 쓰거나.. 등등)
    - 예를 들어, 주민등록번호도 기본 키로 적절하지 않다.
        - 예시 : 개인정보보호법 변경으로, 주민번호를 더 이상 저장하면 안 된다
        → 근데 PK로 주민번호 썼으면?? → 자연키는 PK로 적절하지 않다!

    - 권장 : Long형 + 대체 키 + 키 생성전략 사용

- IDENTITY (DB에 위임) 하는 경우에는 왜 바로 em.persist() 시점에 바로 INSERT 실행하는가?
    - 영속성 컨텍스트 관리 위해서는, KEY(PK)와 VALUE (Object) 필요
    - 그런데 PK 생성을 DB에 위임하면, Cache 위해서 필요한 Key를 알 수 없음!
    - 어쩔 수 없이, 저장(persist) 하는 시점에 바로 INSERT 해야 한다.
    - 다시 말해서, IDENTITY 사용시 Buffering 전략을 쓸 수 없다!

- SEQUENCE 방식의 경우
    - SEQUENCE만 받아서 오고 (PK), 실제로 INSERT는 Commit 시점까지 미룰 수 있다.
    - 하지만 SEQUENCE 번호를 받아오는 건 Network 사용해야 한다! → 성능 문제 발생될 수 있다
    - 해결책 : initialValue, allocationSize (SEQUENCE, TABLE 전략 둘 다에서 사용)
        - DB에 한번 받아올 때 allocationSize만큼 (예시 : 50개) 를 미리 받아온다!
        - 50개 다 사용하기 전까지는 내부에서 메모리에서 1씩 사용한다
        - 다 썼으면 다시 50개 할당받아 온다!
        - 미리 값을 올려두는 방식이므로, 동시성 등 문제가 없다.
