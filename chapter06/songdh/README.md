# Item 34 int 상수 대신 열거타입을 사용하라

### 열거타입 지원하기 전에 사용하던 방법
#### 정수 열거 패턴 ^item34
- 타입 안전을 보장할 방법이 없음
- 표현력 좋지 않음
- 컴파일시 값이 클라이언트 파일에 그대로 새겨지므로 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야하여 깨지기 쉬움
- 문자열로 출력하기 까다롭고 의미 파악이 어려움
- 상수의 개수, 순회를 하기 어려움
#### 문자열 열거 패턴
- 상수의 의미출력 가능
- 프로그래머가 상수의 이름대신 문자열 값을 하드코딩하게 만들어 좋지 앟음
- 오타로 인한 런타임 버그 발생 가능성이 있음
- 문자열 비교로 인한 성능 저하
### 열거타입
- 완전한 형태의 클래스
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개
- 밖에서 접근할 수 있는 생성자를 공개하지 않아 final
- 열거타입 선언으로 만들어진 인스턴스들은 하나씩만 존재함이 보장 => 인스턴스 통제
- 싱글턴은 원소가 하나인 열거타입, 열거타입은 싱글턴을 일반화한 형태
- 컴파일 타입 안전성제공
- 각자의 이름공간이 존재하여 이름이 같은 상수도 공존 가능
-  새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 됨
-  toString 메서드는 출력하기 적합한 문자열 제공
- 정수 열거 패턴의 단점 해소됨
#### 장점
- 정의된 상수의 값을 배열에 담아 반환하는 정적 메서드 values제공
- toSteing 메서드는 상수이름을 문자열로 반환
- 상수 제거시
	- 참조하지 않을 경우 영향 없음
	- 참조할 경우
		- 컴파일 시 디버깅에 유용항 메세지가 든 컴파일 오류 발생
		- 컴파일 하지 않은 경우 런타임시 유용한 예외가 발생
	- private 나 package-private 메서드로 구현시 자신을 선언한 클래스 혹은 패키지에서만 사용할 수 있는 기능 제공
	
#### 필드
- 각 상수와 연관된 데이터를 해당 상수자체에 내재
#### 메서드
- 고차원의 추상개념을 표현할 수 있음
- 상수마다 동작이 달라져야하는 상황의 경우 열거타입에 추상 메서드 선언하고 각 클래스 몸체에서 각  상수에 알맞게 재정의 => 상수별 메서드 구현
	- 재정의 하지 않은 경우 컴파일 오류
	- 상수별 데이터와 결합 가능
	- 단점 
		- 열거타입 상수끼리 코드를 공유하기 어려윰
		- switch 문을 사용하면 쉽게 가능
			- 관리관점에서 위험
		- 열거타입 내부에서 구현하지 않게 변경
			- 새로운 열거타입을 생성하여 내부에 각각의 함수를 선언하고
			- 인자로 함수가 선언된 열거타입을 받기
			- 전략 열거타입 패턴
		- 기존 열거타입에 상수별 동작을 혼합해 넣을 때는 switch가 좋은 선택일 수 있음
			- 추가하려는 메서드가 의미상 열거타입에속하지 않으면 직접 만든 열거타입이라도 이 방식을 적용하는게 좋음

열거타입 상수를 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장

#### fromString 메서드 제공 고려
```java
private static final Map<String, Operation> stringToEnum = 
	Stream.of(values()).collect(
		toMap(Obejct::toString, e -> e)); 
//Optional로 반환하여 값이 존재하지않을 상황을 클라이언트에게 알린다. 
public static Optional<Operation> fromString(String symbol) { 
	return Optional.ofNullable(stringToEnum.get(symbol)); 
}
```
toString이 반환하는 문자열을 해당 열거타입 상수로 변환해주는 역할
상수의 문자열 표현이 고유해야
Optional 반환 문자열이 가리키는 연산이 존재하니 않을 수 있음을 클라이언트에 알림 그리고 상황을 대처할 수 있도록 함
#### 열거타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없음
- 이 방식이 허용되면 런타임에 NullPointerException이 발생
- 열거타입의 생성자에 접근할수 잇는것은 상수변수
- 열거타입 생성자에서 같은 열거타입의 다른 상수에도 접근 할 수 업음=> 다른 형제 상수도 statkc이므로 열거타임 생성자에서 정적 필드에 접근할 수 없다는 제약 적용

열거타임을 써야할 때
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수집합
열거타입에 정의된 상수 가수가 고정 불변일 필요는 없음

# Item 35 ordinal 메서드 대신 인스턴스 필드를 사용하라

## ordinal 메서드
열거타입에서 제공하는 메서드
해당 상수가 열거타입에서 몇번째 위치인지 반환
#### 단점
- 유지보수 하기가 쉽지 않음
	- 선언 순서를 바꾸는 순간 문제 발생
- 값을 중간에 비워 둘 수 없음

> [!note] ordinal 메서드는 언제?
> 열거타입 상수에 연결된 값은 ordinal이 아닌 인스턴스 필드에서 얻을 수 있도록 하자
> 열거타입 기반의 범용 자료구조에 쓸 목적으로 설계되었으므로 이외에는 쓰지 말 것


# Item 36 비트 필드 대신 EnumSet을 사용하라

## 비트필드 사용
- 열거값들이 집합으로 사용될 경우 각 상수에 서로 다른 2의 거듭 제곱 값을 할당한 정수 열거 패턴([Item34](#^item34))를 사용
- 비트별 OR을 사용해 집합으로 만들어 집합연산을 수행
- 정수 열거 상수의 단점을 지님
- 비트 필드값이 그대로 출력되면 단순한 정수 열거 상수를 출력할때보다 해석하기가 훨씬 어려움
- 최대 몇 비트가 필요한지 API 작성시 미리 예측하여 적절한 타입을 선택해야함

## java.util 패키지의 EnumSet 클래스
- 열거타입의 상수값으로 구성된 집합을 효과적으로 표현
- 내부는 비트 백터로 구현되어 대부분의 경우 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여줌
- 대량작업은 비트를 효율적으로 처리할 수 있는 산술 연산을 써서 구현
- 비트를 직접 다룰 때 겪는 오류들을 EnumSet이 처리해주어 겪지 않게 해줌

# Item 37 ordinal 인덱싱 대신 EnumMap을 사용하라
배열이나 리스트에서 원소를 꺼낼때 ordinal 메서드로 인덱스를 얻는 코드가 있다 
정수는 열거타입과 달리 타입 안전하지 않기 때문에 정확한 정수 값을 사용해야한다는 것을 보증해야하는 문제가 있다

## EnumMap
열거타입을 키로 사용하도록 설계한 Map 구현체
#### 장점
- 안전하지 않은 형변환은 쓰지 않고 맵의 키인 열거타입이 출력용 문자열을 제공하여 출력결과에 직접 레이블을 달지 않아도 됨
- 베열 인덱스를 계산하는 과정에서 오류가 날 가능성도 봉쇄됨
- 내부에서 배열을 사용하기 때문에 ordinal을 쓴 배열에 비견됨
- 내부 구현 방식을 안을 숨겨서 Map의 타입 안전성과 배열의 성능을 얻음
- EmunMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입정보 제공

EnumMap이 아닌 맵 구현체를 사용한다면 enumMap을 사용해서 얻은 공간과 성능 이점이 감소함

## 두 열거타입의 값을 맵핑하는 경우
#### 사람이 직접 2차원 배열로 만든 경우
```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT,FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        private static final Transition[][] TRANSITIONS = {
                {null, MELT, SUBLIME},
                {FREEZE, null, BOIL},
                {DEPOSIT, CONDENSE, null}
        };

        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
컴파일러가 ordinal과 배열 인덱스의 관계를 알지 못함
열거타입을 수정하면서 배열을 수정하지 않거나 잘못 수정한다면 런타임 오류 발생
가짓수가 늘어나면 제곱해서 커지고 null도 늘어남으로 복잡해짐

#### EnumMap을 사용하여 초기화 코드 로직으로 초기화한 경우
```java
public enum Phase {
	SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }
		// 상전이 맵 초기화
        private static final Map<Phase, Map<Phase, Transition>> m =
		    Stream.of(values()).collect(
				groupingBy(t -> t.from, 
					() -> new EnumMap<>(Phase.class), 
				toMap(t -> t.to, t -> t, 
					(x, y) -> y, () -> new EnumMap<>(Phase.class))));
		    
		public static Transition from(Phase from, Phase to) {
			 return m.get(from).get(to);
		}
	}
}
```
새로운 상태를 추가하려면  Enum 객체 목록에 객체를 추가하는 것 만으로 간단하게 끝낼 수 있음
내부가 배열로 구현되어 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 쉬움

# Item 38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

대부분의 경우 열거타입을 확장하는 것은 좋지 못함
- 확장할 수 있는 열거 타입이 어울리는 쓰임
	- 연산코드(연산코드의 각 원소는 특정 기계가 수행하는 연산)

## 열거타입으로 확장 효과를 내는 법
인터페이스를 구현할 수 있다는 사실을 이용하여 연산코드용 안터페이스를 정의하고 열거타입이 이 인터페이스를 구현하게 함
```java
public interface Operation{
	double apply(double x, double y);
}

public enum BasicOperation implements Operation{
	PLUS("+"){
			public double apply(double x, double y){ return x + y;}
	},
	MINUS("-"){
			public double apply(double x, double y){ return x - y;}
	},
	TIMES("*"){
			public double apply(double x, double y){ return x * y;}
	},
	DIVIDE("/"){
			public double apply(double x, double y){ return x / y;}
	}

	private final String symbol;

	BasicOperation(String symbol){
		this.symbol = symbol;
	}

	@Override public String toString(){
		return symbol;
	}
}

public enum ExtendedOperation implements Operation{
	EXP("^") {
		public dobule apply(double x, double y{ return Math.pow(x,y);}
	},	
	REMAINER("%") {
		public dobule apply(double x, double y{ return x % y;}
	};

	private final String symbol;

	ExtendsOperation(String symbol){
		 this.symbol = symbol;
	}

	@Override public String toString(){
		return symbol;
	}
}	
```
- 열거타입이 그 인터페이스의 표준 구현체 역할을 함
-  인터페이스를 연산의 타입으로 사용
-  타입 수준에서도 기본 열거타입 대신 확장된 열거타입을 넘겨 확장된 열거타입의 원소 모두를 사용할 수도 있게 함
```java
1. 클래스 리터럴을 넘겨 확장된 연산이 무엇인지 알려주는 방법
private static <T extends Enum<T> & Operation> void test( 
	Class<T> opEnumType, double x, double y) { 
	for (Operation op : opEnumType.getEnumConstants()) 
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y)); 
}

2. class객체대신 한정적 와일드카드 타입을 넘기는 방법
	1. 여러 구현 타입의 연산을 조합해 호출 할 수 있음
	2. 특정 연산에서 enumSet과 enumMap을 사용하지 못함
private static void test(Collection<? extends Operation> opSet, double x, double y) {
	for (Operation op : opSet) 
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y)); 
}
```
#### 문제점
열거타입끼리 구현을 상속할 수 없음
- 아무 상태에도 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있음


공유하는 기능이 많으면 별도 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수도 있음

# Item 39 명명 패턴 보다 애너테이션을 사용하라

## 명명패턴
#### 단점
1. 오타가 나면 안된다
2. 올바른 프로그램 요소에서만 사용되라라 보증할 방법이 없다
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다

애너테이션은 앞에서의 문제를 해결해줌

## 애너테이션 생성 예시

 애너테이션 선언할 때 메타애너테이션을 달아줌
@Target은 애너테이션이 적용 가능한 대상을 지정하는데 사용
@Retention은 애너테이션이 유지되는 범위를 지정하는데 사용

```java
/** 
* 테스트 메서드임을 선언하는 애너테이션이다. 
* 매개변수 없는 정적 메서드 전용 -> 제약을 강제하려면 처리기를 구현해야함 javax.annotation.processing API 문서 참고
*/ 
@Retention(RetentionPolicy.RUNTIME) // 런타임에도 @Test 가 유지 없으면 테스트 도구가 인식불가
@Target(ElementType.METHOD) // 메서드 선언에서만 사용되어야함을 알려줌
public @interface Test{}
```
이 애너테이션은 매개면수 없이 단순이 대상에 마킹하는 애너테이션으로 마커애너테이션이라고함
이 클래스에 의미에는 직접 영향을 주지않고 관심있는 프로그램에 추가정보를 제공하여 측별한 처리를 할 기회를 줌

이 테스트 러너는 명령줄로부터 정규화된 클래스 이름을 받아 클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출
```java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(Sample.class.getName());
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) { // 실행할 메서드를 찾아주는 부분
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) { // 테스트 메서드가 예외를 던지면 여기서 잡힘
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + "failed" + exc);
                } catch (Exception exc) { // @Test 애너테이션을 잘못 사용한경우에 잡힘
                    System.out.println("Invalid @Test" + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

#### 애너테이션에 매개변수를 넣는 법
```java
@Retention(RetentionPolicy.RUNTIME) 
@Target(ElementType.METHOD) 
public @interface ExceptionTest { 
	Class<? extends Throwable> value(); 
}

// 사용 방법
@ExceptionTest(ArithmeticException.class) // 매개변수를 넣어준다
public static void m1() { 
	int i = 0; i = i / i; 
}
```
매개변수로 들어온 것만 처리하기 위해서 앞의 코드를 수정

```java
      if (m.isAnnotationPresent(ExceptionTest.class)) {
        tests++;
        try {
          m.invoke(null);
          System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
        } catch (InvocationTargetException wrappedEx) {
          Throwable exc = wrappedEx.getCause();
          Class<? extends Throwable> excType =
              m.getAnnotation(ExceptionTest.class).value(); // 매개변수의 값을 추출하는 부분
          if (excType.isInstance(exc)) {
            passed++;
          } else {
            System.out.printf(
                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                m, excType.getName(), exc);
          }
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @ExceptionTest: " + m);
        }
      }
```
#### 여러개의 변수를 넣는 법 1
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
	Class<? extends Throwable>[] value(); // 배열로 수정
}

// 사용법
@ExceptionTest({ IndexOutOfBoundsException.class,
                NullPointerException.class }) // 원소를 중괄호로 감싸고 쉼표로 구분
public static void doubleBad() {
    ...
}
```

#### 여러개의 변수를 넣는 법 2
매타 애너테이션인 @Repeatable를 사용
- 반복해서 붙일 수 있는 애너테이션을 정의할 때 사용
- 주의점
	1. @Repeatable을 단 애너테이션을 반환하는 '컨테이너 애너테이션'을 하나더 정의하고 @Repeatable에 컨테이너 애너테이션에 class객체를 매개변수로 전달해야 함
	2. 컨테이션 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해아 함 이 컨테이너 애너테이션 타입에는 적절한 보존정책(@Retention)과 적용대상(@Target)을 명시해야 컴파일됨
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

// 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}

// 사용
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class) 
public static void doubleBad() {
    ...
}
```
@Repeatable를 사용시 주의점
- 이 애너테이션을 어러개 달면 하나만 달았을때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용됨
- getAnnotaationsByType메서드는 이 둘을 구분하지 않고 isAnnotationPresent메서드는 둘을 구분
- 메서드에  isAnnotationPresnt로 검사시
	- 반복 가능 애너테이션을 여러 개 단 다음 반복 가능 애너테이션이 달렸는지 검사하는 경우 
		- 컨테이너가 적용되었기 때문에 그렇지 않다고 알려주어 이 메서드들을 모두 무시
	- 반복 가능 애너테이션을 한 개 만 단 다음 컨테이너가 달렸는지 검사한 경우
		-  반복 가능 애너테이션이 적용되었기 때문에 그렇지 않다고 알려주어 이 메서드들을 모두 무시
	- 모두 검사하기 위해서는 따로따로 확인해야함

>애너테이션으로 할수있는 일을 명명패턴으로 처리할 이유가 없음
>도구 제작자를 제외하면 애너테이션 타입을 직접 정의할 일은 거의 없지만 도구 제작자가 아니더라도 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다.
# Item 40 @Override 애너테이션을 일관되게 사용하라

## @Overrride 애너테이션
- 상위 타입의 메서드를 재정의 했음을 뜻함
- 메서드 선언에만 달수 있음
- 일관되게 사용시 악명높은 버그 예방 가능
	- 시그니처가 달라 매서드 재정의가 아닌 메서드 정의를 해 버린 상황
#### 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Overrride 애너테이션을 달자
- 예외 구체 클래스에서 상위클래스의 추상 메서드를 재정의 하는 경우
- 구현하지 않은 추상 메서드가 있다면 컴파일러가 알려주기 때문
대부분의 IDE는 재정의할 메서드를 선택시 @Overrride를 자동으로 붙여줌

#### @Overrride 는 인터페이스의 메서드를 재정의할때도 사용가능
디폴트 메서드를 지원하기 시작하면서 인터페이스 메서드를 구현한 메서드에도 @Overrride를 다는 습관을 들이면 시그니처가 올바른지 확신할 수 있음
# Item 41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커 인터페이스
자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
Serializable이 좋은 예시 => 자신을 구현한 클래스의 인스턴스는 직렬화 할 수 있음을 알려줌

## 마커인터페이스와 마커 애너테이션 비교
#### 마커인터페이스가 나은점
1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 사용 가능
	1. 마커 인터페이스는 어엿한 타입이기 때문에 사용했다면 런타임에 발견될 오류를 컴파일 타임에 잡을 수 있음
2. 적용 대상을 정밀하게 지정할 수 있음
	-  애너테이션은 모든 대상에 달 수 있음 부착 타입을 세밀하게 제한하지 못함
	- 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커를 인터페이스로 정의했다면 마킹하고 싶은 클래스에서만 인터페이스를 구현하면 됨 마킹된 타입은 자동으로 인터페이스의 하위 타입임이 보장
	- 마커 인터페스는 객체의 특정 부분을불변식으로 규정하거나 그 타입의 인스턴스른 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도로 사용할 수 있을것

#### 마커 애너테이션이 나은점
- 거대한 애너테이션 시스템의 지원을 받는다는 점
- 애너테이션을 적극 활용하는 프레임워크에서는 마크 애너테이션를 쓰는 쪽이 일관성을 지키는데 좋음

#### 둘 중 뭘 사용해야 할 지 고민된다면
클래스와 인터페이스 외의 프로그램 요소에 마킹 해야할때 -> 애너테이션
마킹된 객체를 매개변수로 작성할 일이 있는 케이스 -> 마커 인터페이스
- 컴파일 타임에 오류를 잡아낼 수 있기 때문
- 이런 일이 절대 없다고 확신하면 마커 애너테이션
애너테이션를 활발히 사용하는 프레임워크라면 마커 애너테이션이 좋다

