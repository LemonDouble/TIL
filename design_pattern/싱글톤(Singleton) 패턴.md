# 싱글톤(Singleton) 패턴

배운 내용 (한줄 요약): Singleton 패턴
분류: 디자인 패턴
자료 링크: https://woowacourse.github.io/javable/post/2020-11-07-singleton/
작성일시: 2021년 6월 28일 오후 1:54

## 싱글톤 패턴 : 객체의 인스턴스를 하나만 생성되도록 하고, 생성된 객체를 어디서든 참고할 수 있도록 하는 패턴

### 1) 왜 싱글톤 패턴을 사용하는가?

1. 메모리 관리 측면에서 이득 

    (한번의 new 연산자를 통해 고정된 메모리를 사용하기 때문에, 추후 해당객체에 접근할 때 메모리 낭비를 방지할 수 있음)

2. 데이터 공유가 쉬움. 

    (전역 인스턴스이기 때문에, 다른 클래스의 인스턴스들이 접근하여 사용할 수 있음)

3. Domain 관점에서 인스턴스가 한 개만 존재하는것을 보증하고 싶은 경우 (Logger 등..)

### 2) 싱글톤 패턴의 문제점은?

1. SOLID의 "개방-폐쇄 원칙" 에 위배되는 경우가 많음. 
2. 테스트하기 어렵다.
3. Race Condition 관련 문제가 생길 수 있다. (Synchronized 키워드 통해 해결 가능하다)

### 3) 싱글톤 패턴의 구현

- Bill Pugh Singleton Implementation

```java
public static Singleton {

	private Singleton(){
		//생성자는 외부에서 호출할 수 없도록 private로 지정
	}

	private static class SingletonHelper{
		private static final Singleton INSTANCE = new Singletion();
	}

	public static Singleton getInstance(){
		return SingletonHelper.INSTANCE;
	}

	public void say(){
		System.out.println("Hello, World!");
	}
}
```

SingletonHelper()는 getInstance()가 호출되었을 때 JVM에 로드되므로, 메모리 누수가 없음.