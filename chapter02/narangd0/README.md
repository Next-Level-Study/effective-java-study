# Chapter 2. 객체 생성과 파괴

## Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

### 클래스의 인스턴스를 얻는 2가지 수단

- public 생성자
- (생성자와 별도) 정적 팩터리 메서드(static factory method)
    
    ```jsx
    public static Boolean valueOf(boolean b) {
    	return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```
    

### 정적 팩터리 메서드 장점

1. 이름을 가질 수 있다
    1. 반환되는 객체의 특성을 설명함으로써 의미 전달이 가능하다
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
    1. 불변 클래스: 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
    2. 플라이웨이트 패턴(Flyweight pattern)과 유사한 기법
    3. 인스턴스 통제 클래스
        1. 반복되는 요청에 같은 객체를 반환 → 인스턴스의 생애 주기를 통제 가능
        2. 인스턴스 통제 이유?
        클래스를 싱글턴, 인스턴스화 불가 클래스로 만들 수 있다.
        3. 플라이웨이트 패턴, 열거 타입
3. 반환 타입의 하위 타입 객체 반환이 가능하다.
    1. 반환할 객체의 클래스를 자유롭게 선택이 가능한 유연성 초래
    2. 인터페이스를 정적 펙터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크에 활용되는 기술
    3. 자바 8: 인터페이스가 public 정적 멤버를 가질 수 있다.
    자바 9: 인터페이스가 private 정적 메서드를 가질 수 있다.
4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    1. 서비스 제공자 프레임워크의 근간이 되는 유연함 제공, 예) JDBC
    2. === 서비스 제공자 프레임워크 패턴 ===

### 정적 펙토리 메서드 단점

1. 정적 펙토리 메서드만 제공 시 하위 클래스를 만들 수 없다.
    1. 상속보다 컴포지션을 사용하도록 하고, 불변 타입으로 만드려면 이 제약을 지켜야 하기 때문에 오히려 장점으로 본다는 내용도 있다.
2. 프로그래머가 찾기 어렵다.
    1. API 문서 작성과 규약을 따라 메서드 이름을 짓자.
    2. === 정적 펙토리 메서드 명명 방식 === p.12

## Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 선택적 매개변수가 많을 때

1. 점층적 생성자 패턴(telescoping constructor pattern)
    1. 필수 매개변수만 받는 생성자, … 모든 선택 매개변수를 받는 생성자까지…
2. 자바빈즈 패턴(JavaBeans pattern) 
    1. 매개변수가 없는 생성자로 객체 생성 → setter 메서드를 통해 원하는 매개변수의 값 설정
    2. But
        1. 객체를 하나 만드려면 여러 메서드를 호출해야 한다.
        2. 객체가 완전히 생성되기 전까지 일관성(consistency)가 무너진 상태에 놓인다.
        3. 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 추가 작업 필요
3. 빌더 패턴(Builder parttern)
    1. 필수 매개변수만으로 생성자를 호출하여 빌더 객체를 얻는다 → 일종의 세터 메서드를 통해 원하는 선택 매개변수를 설정한다. → build 메서드를 통해 객체를 얻는다.
    2. === 빌더 패턴 === p.17

## Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

### 싱글턴(singleton)

인스턴스를 오직 하나만 생성할 수 있는 클래스

1. public static final 필드 방식
    
    ```jsx
    public class Elvis {
    	**public** static final Elvis INSTANCE = new Elvis();
    	private Elvis() {}
    }
    ```
    
2. 정적 펙토리 방식
    
    ```jsx
    public class Elvis {
    	**private** static final Elvis INSTANCE = new Elvis();
    	private Elvis() {}
    	public static Elvis getInstance() { return INSTANCE; }
    }
    ```
    
3. 열거 타입 방식 - best
    
    ```jsx
    public enum Elvis {
    	INSTANCE;
    }
    ```
    

## Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

### 정적 메서드와 정적 필드만 담은 클래스를 만들 때

- 예) java.lang.Math, java.util.Arrays, java.util.Collections
- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다. 이때, 그렇다고 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만든다.
- private 생성자를 추가하여 클래스의 인스턴스화를 막자!
    
    ```jsx
    public class UtilityClass {
    	// 기본 생성자가 만들어지는 것을 방지!
    	private UtilityClass() {
    		throw new AssertionError();
    	}
    }
    ```
    
- private 생성자를 사용함으로써, 상속을 불가능하게 만드는 효과도 있다.

## Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

### 사용하는 자원에 따라 동작이 달라지는 클래스

- 자원을 교체하는 메서드 추가? → 오류 발생이 쉽고 멀티 스레드 환경에서 쓸 수 없다
- 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 클래스가 여러 인스턴스를 지원하고, 클라이언트가 원하는 자원을 사용할 수 있어야 한다.
- 의존 객체 주입: 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하자

```jsx
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellCheck(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}
```

- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용이 가능하다.
- 의존성 주입 프레임워크: 스프링(Spring), 대거(Dagger), 주스(Guice)
- === 의존성 주입을 하는 이유 ===
    
    프로그램 디자인이 **결합도를 느슨**하게 되도록하고 의존관계 역전 원칙과 단일 책임 원칙을 따르도록 클라이언트의 생성에 대한 의존성을 클라이언트의 행위로부터 분리하는 것
    

## Item 6. 불필요한 객체 생성을 피하라

### 동일한 기능의 객체를 매번 생성하지 말고 객체 하나를 재사용 하자

```jsx
(1) String s = new String("bikini");
```

→ 기능적으로 동일한 String 인스턴스가 쓸데없이 생성된다.

```jsx
(2) String s = "bikini";
```

→ 새로운 인스턴스를 매번 생성하는 대신 하나의 인스턴스를 사용한다. (똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장됨)

- Item 1의 생성자 대신 정적 팩터리 메서드를 사용하면 불필요한 객체 생성을 피할 수 있다.

### 생성 비용이 비싼 객체

- 비싼 객체 사용이 반복적으로 필요하다면 캐싱하여 재사용하자.
- 정규표현식용 Pattern 인스턴스: 입력받은 정규표현식인 유한 상태 머신(finite state machine)을 만들기 때문에 인스턴스 생성 비용이 높다.
- 개선 방법?
    
    ```jsx
    public class RomanNumerals {
    	private static final Pattern ROMAN = Pattern.compile("...");
    
    	static boolean isRomanNumeral(String s) {
    		return ROMAN.matcher(s).matches();
    	}
    }
    ```
    
    - 클래스 초기화 시 Pattern 인스턴스를 생성하여 캐싱 후, 메서드가 호출될 때 이 인스턴스를 재사용하도록 만든다.

### 오토 박싱

프로그래머가 기본 타입(primitive type) 과 박싱된 기본 타입(reference type)을 섞어 쓸 때 자동으로 상호 변환해주는 기술

```jsx
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;
	return sum;
}
```

박싱된 기본 타입보다는 기본 타입을 사용하고, 의도하지 않은 오토박싱에 주의하자.

=== (Item 50) 방어적 복사가 필요한 상황에서의 객체 재사용 vs 필요 없는 객체를 반복 생성 ===

## Item 7. 다 쓴 객체 참조를 해제하라

자바의 가비지 컬렉터를 믿고 메모리 관리에 신경쓰지 않아도 된다는 것은 오해이다.

### 메모리 누수

- 메모리 누수 → 가비지 컬렉션 활동과 메모리 사용량 증가로 인한 성능 저하, 심할 시 디스크 페이징과 OutOfMemoryError 발생…
- 가비지 컬렉션 - 객체 참조 하나를 살려두면, gc는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수하지 못한다.
- 해결? 해당 참조를 다 썼을 때 null 처리(참조 해제)하자!

```jsx
public Object pop() {
	if (size == 0)
		throw new EmptyStackException();
	Object result = elements[--size];
	elements[size] = null;  // 다 썼으니, 참조 해제!
	return result;
}
```

- === WeakHashMap을 활용하여 캐시 만들기 ===

## Item 8. finalizer와 cleaner 사용을 피하라

자바에서 제공하는 두 가지 객체 소멸자, finalizer, cleaner

- 예측이 불가능, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- 객체에 접근할 수 없게 된 후, 이 소멸자가 실행되기까지 얼마나 걸릴지 알 수 없다.
- 즉 제때 실행되어야 하는 작업 실행이 불가능하다.
- 해결?
    - AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 close() 메서드를 호출한다. (일반적으로, 예외가 발생해도 제대로 종료될 수 있도록 try-with-resources를 사용하자.)

## Item 9. try-finally보다는 try-with-resources를 사용하라

### try-with-resources

- 코드가 간결하고, 읽기 수월할 뿐 아니라 문제 진단에도 훨씬 좋다. 그냥 쓰자
- 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.

---

플라이웨이트 패턴

싱글톤 패턴 구현할 때 public 필드, private 필드 방식 차이점?
