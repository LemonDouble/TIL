# JAVA : Collections 프레임워크

분류: JAVA
작성일시: 2021년 7월 29일 오전 10:10

## 1. 컬렉션 프레임워크란?

- 널리 알려진 자료구조를 바탕으로, 객체들을 효율적으로 추가,삭제, 검색할 수 있도록 java.util에 관련 인터페이스와 클래스를 추가해 놓은 것
- List Interface : 순서를 유지/저장, 중복 저장 가능
    - ArrayList
    - Vector
    - LinkedList
- Set Interface : 순서를 유지하지 않고 저장, 중복 저장 불가
    - HashSet
    - TreeSet
- Map Interface 키와 값의 쌍으로 저장, 키는 중복 안 됨
    - HashMap
    - HashTable
    - TreeMap
    - Properties

이 있다.

## 2. List 컬렉션

- 배열처럼, 객체를 일렬로 늘어놓은 구조, 객체를 인덱스로 관리하므로, 객체를 저장하면 자동으로 인덱스가 부여되고 인덱스로 객체를 검색/삭제가 가능.
- List 컬렉션은 객체 자체를 저장하는 것이 아닌, 객체의 Reference를 저장.
- 중복 객체를 저장할 경우, 동일한 Reference를 저장

- Interface Method
    - 객체 추가 / 수정
        - boolean add(E e) : 주어진 객체를 맨 끝에 추가
        - void add(int idnex, E element) : 주어진 인덱스에 객체를 추가
        - E set(int index, E element) : 주어진 인덱스에 저장된 객체를 새 객체로 변경
    - 객체 검색
        - boolean contains(Object o) : 주어진 객체가 저장되어 있는지를 반환
        - E get(int index) : 주어진 인덱스에 저장된 객체를 리턴
        - boolean isEmpty() : 컬렉션이 비어 있는지 확인
        - int size() : 저장된 모든 객체의 수를 리턴
    - 객체 삭제
        - void clear() : 저장된 모든 객체를 삭제
        - E remove(int index) : 주어진 인덱스에 저장된 객체를 삭제
        - boolean Remove(Object o) 주어진 객체를 삭제

- ArrayList 구현 클래스
    - 배열과 비슷하지만, Capacity를 초과하면 자동으로 Capacity가 늘어난다.
    - Index 검색, 가장 마지막 Index에 Object 추가가 빈번한 경우 사용하면 좋은 성능 발휘
    - 하지만 빈번한 삽입/삭제 일어나는 경우 성능이 좋지 않다.
    - 특정 Index Object 삭제되면, 그 뒤에 있는 Object 한칸씩 땡겨오는 연산 발생
    - 저장시 Index 0부터 차례로 Object를 저장

- Vector 구현 클래스
    - Synchronized 메소드로 구성, 멀티스레드가 동시에 이 메소드를 실행할 수 없다.
    - 하나의 메소드가 완료되어야만 다른 스레드를 실행할 수 있다.
    - 멀티 스레드 환경에서도 안전하게 객체를 추가/삭제 가능
    - 즉, 위와 같으므로 Thread Safe!

- ArrayList vs Vector
    - Single Thread 사용시 : Synchronized 되지 않으므로 ArrayList가 더 빠르다.
    - ArrayList는 Capacity 초과시 50% 를 확장시키지만, Vector는 100% 증가시킨다.

- LinkedList 구현 클래스
    - 우리가 알고 있는 양방향 Linked List
    - 빈번한 삽입 / 삭제시 성능이 좋다.
    - 순차적으로 추가/삭제하는 경우, 잦은 검색이 일어나는 경우 검색이 느리다.

## 3. Set 컬렉션

- 저장 순서가 유지되지 않음
- 수학에서의 집합과 유사!

- Interface Method
    - 객체 추가
        - boolean add(E e) : 주어진 객체를 저장, 중복이면 false return
    - 객체 검색
        - boolean contains(Object o) : 주어진 객체가 저장되어 있는지 여부
        - boolean isEmpty() : 컬렉션이 비어 있는지 조사
        - Iterator<E> iterator() : 저장된 객체를 한 번씩 가져오는 반복자(Iterator) return
        - int size() : 저장되어 있는 전체 객체 수 리턴
    - 객체 삭제
        - void clear() : 저장된 모든 객체를 삭제
        - boolean remove(Object o) : 주어진 객체를 삭제
    - Iterator Interface Method
        - boolean hasNext() : 다음 객체 있으면 true, 없으면 false를 return
        - E next() : Collections에서 객체 하나를 가져옴
        - void remove() : Set Collection에서 객체를 제거한다.

        ```java
        while(iterator.hasNext()){
        	Sting str = iterator.next();
        	
        	if(str.equals("홍길동"){
        		iterator.remove();
        	}
        }
        ```

- HashSet 구현 클래스
    - 각 객체의 hashCode() 메소드를 먼저 호출하여 저장된 객체들의 HashCode와 비교
    - 만일 동일한 객체가 있다면, equals() 메소드로 두 객체를 비교
    - equals() 까지 같아면, 동일한 객체로 판단, 중복 저장하지 않음

    - hashCode (String) : 다른 문자열인데 같은 값 Return 될 수 있음!
        - 예를 들어, Siblings와 Teheran은 같은 hashCode 리턴한다.
        - 따라서 hashCode가 같다는게 서로 같은 객체임을 보장하진 않는다!
        - TIP : String name, int age 있는 class에서 hashCode를 Override 할 경우, name.hashCode() + age 식으로 오버라이드 할 수 있다.

- TreeSet 구현 클래스
    - Red-Black Tree를 기반으로 한, 트리 기반 구현체
    - TreeSet Class로 선언하면 (TreeSet<String> treeset = new Treeset) 추가 검색/정렬 메소드를 사용할 수 있음.
        - 검색 메소드
            - E first() : 최소값 리턴
            - E last() : 최대값 리턴
            - E lower(E e) : 주어진 객체보다 바로 작은 객체를 출력, 없으면 null
            - E higher(E e) : 주어진 객체보다 바로 큰 객체를 출력, 없으면 null
            - E floor(E e) : 주어진 객체와 동등한 객체가 있으면 리턴, 없다면 주어진 객체의 바로 작은객체를 리턴
            - E ceiling(E e) : 주어진 객체와 동등한 객체가 있으면 리턴, 없다면 주어진 객체의 바로 큰 객체를 리턴
            - E pollFirst() : 제일 작은 객체를 꺼내오고 컬렉션에서 제거
            - E pollLast() : 제일 큰 객체를 꺼내오고 컬렉션에서 제거
        - 정렬 메소드
            - Iterator<E> descendingIterator() : 내림차순으로 정렬된 Iterator 리턴
            - NavigableSet<E> decendingSet() : 내림차순으로 정렬된 NavigableSet 리턴
            - 오름차순으로 정렬하고 싶다면 descendingSet 두번 호출!

            ```java
            NavigableSet<E> descendingSet = treeSet.descendingSet();
            NavigableSet<E> ascendingSet = descendingSet.descendingSet();
            ```

        - 범위 검색 메소드
            - NavigableSet<E> headSet(E toElement, boolean inclusive)
            : 주어진 객체보다 작은 객체들을 NavigableSet으로 리턴, 주어진 객체의 포함 여부는  두 번째 Parameter에 의해 결정
            - NavigableSet<E> tailSet(E toElement, boolean inclusive)
            :주어진 객체보다 큰 객체들을 NavigableSet으로 리턴, 주어진 객체의 포함 여부는  두 번째 Parameter에 의해 결정

## 4. Map 컬렉션

- Entry 객체 (Key, Value로 구성) 을 저장
- Key, Value는 모두 객체
- Key는 중복 불가, 하지만 Value는 중복 가능

- Interface Method ( K : key, V : value)
    - 객체 추가
        - V put(K key, V value) : 주어진 키로 값을 저장. 새로운 키일 경우 null 리턴, 동일한 키가 있을 경우 값을 대체하고 이전 값을 리턴
    - 객체 검색
        - boolean containsKey(Object key) : 주어진 키가 있는지 여부를 리턴
        - boolean containsValue(Object value) : 주어진 값이 있는지 여부를 리턴
        - Set<Map.Entry<K,V>> entrySet(): 키와 값의 쌍으로 구성된 모든 Map.Entry 객체를 Set에 담아 리턴
        - V get(Object key) : 주어진 키가 있는 값을 리턴
        - boolean isEmpty() : 컬렉션이 비어 있는지 여부
        - Set<K> keySet() : 모든 키를 Set 객체에 담아서 리턴
        - int size() : 저장된 키의 총 수를 리턴
        - Collection<V> values() : 저장된 모든 값을 Collection에 담아서 리턴
    - 객체 삭제
        - void clear() : 모든 Map.Entry(키와 값) 을 삭제
        - V remove(Obejct key) : 주어진 키와 일치하는 Map.Entry를 삭제하고 값을 리턴

- HashMap 구현 클래스
    - HashMap의 Key로 사용할 객체는, hashCode()와 equals() 메소드를 override해줘야 한다.

- Hashtable 구현 클래스
    - 마찬가지로 Key 객체는 hashCode(), equals() override 해 줘야 한다.
    - HashMap과 같은 내부 구조지만, Synchronized 메소드로 구성되어 Thread-Safe

- Properties 구현 클래스
    - Hashtable의 하위 클래스, 따라서 Hashtable의 모든 특징을 그대로 가지고 있다
    - Key, Value의 값을 String 타입으로 제한
    - 어플리케이션의 옵션, DB 연결 정보, 국제화 정보가 저장된 ~.properties 파일을 읽을 때 주로 사용

- TreeMap 구현 클래스
    - Treeset과 비슷하나, Map.Entry를 저장

## 5. Comparable과 Comparator

- TreeSet, TreeMap의 경우 Key가 저장과 동시에 자동으로 오름차순으로 정렬됨
- Integer, Double일 경우 값으로 정렬, String일 경우 Unicode로 정렬

- TreeSet, TreeMap 모두 정렬을 위해 java.lang.Comparable 구현 객체를 요구
- 사용자 정의 클래스도 Comparable 클래스를 오버라이드 하면 자동 정렬이 가능
- java.lang.Comparable 내의 compareTo() 메소드를 오버라이드!
    - int compareTo(T o) : 현재 객체가 주어진 객체와 같으면 0, 주어진 객체보다 작으면 음수, 주어진 객체보다 크면 양수를 리턴

    ```java
    //예시 : name, age 있는 Person class를 나이 순으로 정렬

    @Override
    public int compareTo(Person o){
    	if(age < o.age) return -1; //현재 객체가 주어진 객체보다 작으므로 음수 
    	else if (age == o.age) return 0;
    	else return 1; //현재 객체가 주어진 객체보다 크므로 양수
    }
    ```

- TreeSet, TreeMap의 키가 Compareable을 구현하지 않은 경우 ClassCastException이 발생

## 6. Stack, Queue

- Stack
    - 주요 메소드
        - E push(E item) : 주어진 객체를 스택에 넣는다
        - E peek() : 스택의 맨 위 객체를 가져온다. 객체를 스택에서 제거하지 않는다.
        - E pop() : 스택의 맨 위 객체를 가져온다. 객체를 스택에서 제거한다.

- Queue
    - 주요 메소드
        - boolean offer(E e) : 주어진 객체를 넣는다.
        - E peek() : 객체 하나를 가져온다. 객체를 큐에서 제거하지 않는다.
        - E poll() : 객체 하나를 가져온다. 객체를 큐에서 제거한다.

## 7. Synchronized Collections

- Vector, HashTable은 이미 동기화 지원
- 하지만 그 이외의 Collections도, Synchronized 지원해야 할 때 있음
- 따라서 Synchronized 래핑 메소드 제공

- List<T> synchronizedList(List<T> list) : List를 Synchronized된 List로 리턴
- Map<K,V>synchronizedMap(Map<K,V> m) : Map을 Synchronized된 Map으로 리턴
- Set<T> synchronizedSet(Set<T> s) : Set을 Synchronized된 Set으로 리턴

## 8. 병렬 처리를 위한 컬렉션

- Synchronized는 Thread-Safe 하지만 전체 요소를 빠르게 처리하지 못함
- Synchronize의 경우, 하나를 처리하면 전체가 잠금되므로 병렬 처리에는 효율적이지 못함

- 효율적 처리 위해선 ConcurrentHashMap, ConcurrentLikedQueue 사용 가능

- ConcurrentHashMap : 부분 잠금 구현, 현재 처리하고 있는 segment만 잠근다.
- ConcurrentLinkedQueue : Lock-Free 알고리즘 구현