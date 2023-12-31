# Chapter 6. 열거 타입과 애너테이션 (34 ~ 41)

특수 목적의 참조 타입

1. 클래스 일종의 열거 타입
2. 인터페이스 일종의 애너테이션

# 아이템 34 int 상수 대신 열거 타입을 사용하라

### 열거 타입이 지원되기 전

- 정수 열거 패턴(int enum pattern)
- 문자열 열거 패턴(string enum pattern)
- 단점
    1. 타입 안전 보장 X
    2. namespace 미지원으로 접두어 사용이 필수
    3. 문자열 출력이 까다롭다

### 열거 타입(enum type)

- 클래스이다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 싱글턴을 일반화한 형태
- 컴파일타임에 타입 안전성 제공
- 이름공간이 있어 이름이 같은 상수가 공존할 수 있다
- toString 메서드로 적합한 문자열 출력 가능

데이터와 메서드를 갖는 열거 타입

- 열거 타입 상수와 데이터를 연결 지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장한다. 열거 타입은 근본적으로 불변이라 모든는 final 이어야 한다.

values()

- 열거 타입으로 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드, 선언된 순서로 저장된다.

클래스 레벨

- 널리 쓰이는 열거 타입은 톱레벨 클래스로, 특정 톱레벨 클래스에서만 쓰이는 열거 타입은 해당 클래스의 멤버 클래스로 만든다.

상수별로 다르게 동작하는 코드를 구현하고싶다면

1. switch 문
    - 깨지기 쉬운 코드: 새로운 상수가 추가되면 새로운 case문도 추가되어야 한다.
2. 상수별 메서드 구현(constant-specific method implementation)
    - apply 라는 추상 메서드를 선언, 각 상수별 클래스 몸체(constant-specific class body)에 재정의하여 구현한다.

```java
public enum Operation {
	PLUS { public double apply(...) {...} },
	MINUS { public double apply(...) {...} }

	public abstract double apply(...);
}
```

valueOf(String)

- 상수 이름을 입력 받아 그 이름에 해당하는 상수를 반환해주는 메서드

fromString

- 열거 타입의 toString 이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 메서드
- toString 메서드를 재정의하는 경우에 함께 제공하면 좋다

```java
public enum Operation {
    PLUS("+") { public double apply(double x, double y) { return x + y; } },
    MINUS("-") { public double apply(double x, double y) { return x - y; } };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public abstract double apply(double x, double y);

    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(toMap(Object::toString, e -> e));

    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
		// kotlin 이었으면... Operation? 을 return 했을텐데...
}
```

전략 열거 타입 패턴 - private 중첩 열거 타입 선언

```java
package chapter.chapter06;

public enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY), SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    enum PayType {
        WEEKDAY {
            int overtimePay(int mins, int payRate) {
                return 0;
            }
        }, WEEKEND {
            int overtimePay(int mins, int payRate) {
                return 0;
            }
        };

        abstract int overtimePay(int mins, int payRate);

        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return ...;
        }
    }
}
```

열거 타입은 언제 써?

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 열거 타입을 쓰자.

# 아이템 35 ordinal 메서드 대신 인스턴스 필드를 사용하라

### Ordinal 메서드

대부분의 열거 타입 상수는 하나의 상숫값에 대응되며, 모든 열거 타입은 열거 타입 상수가 몇 번째 위치인지 반환하는 메서드를 제공한다.

단점

- 상수 선언 순서를 바꾸게 되면?
- 이미 사용 중인 정수와 값이 같은 상수를 추가할 수 없다.
- 값을 중간에 비워둘 수 없다. → dummy 상수를 추가해야만 한다.

결론

- 열거 타입 상수와 연결되는 값은 ordinal 메서드를 사용하지 말고, 인스턴스 필드로 저장하자.
- ordinal 메서드는 EnumSet, EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.

# 아이템 36 비트 필드 대신 EnumSet을 사용하라

### 비트 필드(bit field)

열거 값들이 단독이 아니라 집합으로 사용될 경우 예전에는 각 상수에 2의 거듭제곱 값을 할당하는 정수 열거 패턴을 사용했었다.

```java
package chapter.chapter06;

public class Text {
    public static final int STYLE_BOLD          = 1 << 0;
    public static final int STYLE_ITALIC        = 1 << 1;
    public static final int STYLE_UNDERLINE     = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;

    /**
     * @param styles 0 개 이상의 STYLE_ 상수를 비트별 OR 한 값
     */
    public void applyStyles(int styles) {
        // ...
    }
}

// ---

public static void main(String[] args) {
		// 비트 필드를 사용하면 비트별 연산으로 집합 연산(합집합, 교집합)을 효율적으로 수행할 수 있다!
		text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
}
```

단점

- 정수 열거 상수 패턴의 단점을 그대로 가진다.
- 비트 필드 값의 출력 값은 해석하기가 어렵다.
- 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 어렵다.
- 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.

### 상수 집합일 때는, EnumSet 클래스!

내부적으로 비트 벡터로 구현되어있다. 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 가진다.

- 유일한 단점? 불변 EnumSet 을 만들 수 없다.
    - 자바 11까지도 만들 수 없으나 구글의 구아바(Guava) 라이브러리를 사용하면 불변 EnumSet 을 만들 수 있긴 하다.

```java
public class Text2 {
    public enum Style {
        BOLD, ITALIC, UNDERLINE, STRIKETHROUGH;
    }

    // EnumSet 이 가장 적절하다!
    public void applyStyles(Set<Style> styles) {
        // ...
    }
}

// text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

# 아이템 37 ordinal 인덱싱 대신 EnumMap을 사용하라

```java
package chapter.chapter06;

import java.util.Arrays;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class Plant {
    enum LifeCycle {
        ANNUAL, PERENNIAL, BIENNIAL,
    }

    private String name;
    private LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }

        for (Plant p : List.of(new Plant("식물1", LifeCycle.ANNUAL), new Plant("식물2", LifeCycle.PERENNIAL))) {
            plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
        }

        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
}
```

문제

- 배열과 제네릭의 비호환(아이템 26): 비검사 형변환 수행, 깔끔히 컴파일되지 않음
- 배열은 각 인덱스의 의미가 없으므로 출력 결과에 직접 레이블을 달아야 한다.
- 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.
    - 정수(ordinal 메서드 반환값)는 열거 타입과 달리 타입 안전하지 않다.

### EnumMap

위 예제에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다. 열거 타입을 키로 사용하도록 설계한 EnumMap 을 쓰자!

```java
// 열거 타입을 키로 사용하는 EnumMap 활용
Map<LifeCycle, Set<Plant>> plantsByLifeCycle2 = new EnumMap<>(LifeCycle.class);

for (LifeCycle lc : LifeCycle.values()) {
    plantsByLifeCycle2.put(lc, new HashSet<>());
}

for (Plant p : garden) {
    plantsByLifeCycle2.get(p.lifeCycle).add(p);
}

System.out.println(plantsByLifeCycle2);
// ANNUAL=[식물1], PERENNIAL=[식물2], BIENNIAL=[식물3]}
```

비교

- 짧고, 명료하고, 안전하고, 성능도 괜찮다.
    - EnumMap 이 내부에서 배열을 사용하기 때문에 ordinal을 쓴 배열과 비슷하다.
- 맵의 key 인 열거 타입이 그 자체로 출력용 문자열을 가지므로, 출력문에 직접 레이블을 달지 않아도 된다.
- 배열 인덱스 계산 과정에서 오류가 날 가능성을 없앤다.
- new EnumMap<>(LifeCycle.class)
    - 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다. (아이템 33)

정리

- 배열의 인덱스를 얻기 위해 ordinal 을 쓰는 것은 일반적으로 좋지 않다.
- EnumMap 을 사용하자
    - 다차원 관계는 EnumMap<…, EnumMap<…>> 으로 표현한다.

# 아이템 38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

### **확장할 수 없는 열거 타입**

열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern)보다 우수하다. 단, 예외가 하나 있으니, 타입 안전 열거 패턴은 확장할 수 있지만, 열거 타입은 그럴 수 없다는 점이다.

```java
// **확장 시 에러가 발생하는 열거타입**
public enum AEnum {
    A, B, C
}

public enum BEnum extends AEnum { *// enum은 확장할 수 없다는 에러 발생*
    D, E, F
}
```

### **확장형 열거타입에 어울리는 연산코드**

연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다. 기본으로 더하기, 빼기, 곱하기, 나누기 연산을 제공한다고 가정했을 때 제곱, 나머지 연산과 같은 확장된 연산을 제공해야 할 경우가 있다. 그런데 앞서 말했듯 열거타입은 확장이 불가능하다. 하지만 확장의 효과를 내는 방법이 있다. 바로 **인터페이스**를 사용하는 것이다. 기본적인 연산에 대한 열거 타입 클래스를 인터페이스를 사용하여 구현하였다.

```java
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override public double apply(double x, double y) {
            return x+y;
        }
    },
    MINUS("-") {
        @Override public double apply(double x, double y) {
            return x-y;
        }
    },
    TIMES("*") {
        @Override public double apply(double x, double y) {
            return x*y;
        }
    },
    DIVIDE("/") {
        @Override public double apply(double x, double y) {
            return x/y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

### **특별 연산이 추가된다면 인터페이스를 구현하자**

만약 특별 연산이 추가되어야 한다면 새로운 열거 타입 클래스에서 인터페이스를 구현하면 된다.

```java
public enum ExtendedOperation implements Operation {
    EXP("^"){
        @Override public double apply(double x, double y) {
            return Math.pow(x,y);
        }
    },

    REMAINDER("%"){
        @Override public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

### **테스트**

기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용할 수 있다.

```java
public static void main(String[] args) {
    test(ExtendedOperation.class, 3, 3);
}

public static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
	for(Operation op : opEnumType.getEnumConstants()) {
		System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
  }
}
```

- main 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.
- 타입 매개변수 부분인 <T extends Enum<T> & Operation> 코드는 타입 매개변수 T가 Enum<T>. 즉, 열거 타입임과 동시에 Operation의 하위 타입이어야 한다는 것이다. 이는 Enum을 통한 원소 순회와 Operation 인터페이스의 메서드를 호출하기 위함이다.
- 이게 복잡하다면 아래 방법을 사용할 수 있다.

```java
public static void main(String[] args) {
    test(Arrays.asList(ExtendedOperation.values()), 3, 3);
}

public static void test(Collection<? extends Operation> opSet, double x, double y) {
    for(Operation op : opSet) {
        System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x,y));
    }
}
```

- ExtendedOperation의 상수 인스턴스를 List로 만든 후 각각의 상수 인스턴스에서 값을 순회하며 출력하는 방식이다. 이 코드는 위 방법보다 덜 복잡하고 유연해졌다.
- Operation을 구현하는 여러 클래스들에서 본인이 필요한 상수 인스턴스들을 추출하여 List로 넘겨주기만 한다면 다양한 연산들을 호출할 수 있기 때문이다.

### **정리**

- 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입을 만들 수 있다.

# 아이템 39 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴

- 변수나 함수의 이름을 일관된 방식으로 작성하는 패턴을 말한다.
- 이러한 **명명 패턴**은 전통적으로 도구나 프레임워크에서 특별히 다뤄야 할 프로그램 요소에 구분을 위해 사용되어 왔다.
- 예를 들면 JUnit3 에서는 테스트 메서드 이름을 test로 시작하게끔 했다. 이러한 명명 패턴 방식은 효과적이지만 단점도 크다.

### 명명 패턴의 단점

- 오타가 나면 안 된다.
    - 만약 test로 시작되어야 할 메서드 이름이 오타로 인해 tset로 작성되었다면, 명명 패턴에는 벗어나지만 프로그램 상에서는 문제가 없기 때문에 테스트 메서드로 인식하지 못하고 테스트를 수행하지 않는다.
- 명명 패턴을 의도한 곳에서만 사용할 거라는 보장이 없다.
    - 개발자는 JUnit3의 명명 패턴인 **'test'**를 메서드가 아닌 **클래스의 이름**으로 지음으로써 해당 클래스의 모든 테스트 메서드가 수행되길 바랄 수 있다. 하지만 JUnit은 클래스 이름에는 관심이 없다. 따라서 개발자가 의도한 테스트는 전혀 수행되지 않는다.
- 명명 패턴을 적용한 요소를 매개변수로 전달할 마땅한 방법이 없다.
    - 특정 예외를 던져야 성공하는 테스트가 있을 때, 메서드 이름에 포함된 문자열로 예외를 알려주는 방법이 있지만 보기 흉할 뿐 아니라 컴파일러가 문자열이 예외 이름인지 알 도리가 없다.

애너테이션을 사용하면 위의 단점을 모두 해결할 수 있다. 그러므로 많은 단점을 가진 명명 패턴을 사용하기보다는 **애너테이션을 사용하자.**

### 마커 애너테이션

마커 애너테이션은 **아무 매개변수 없이 단순히 대상에 마킹하는 용도**로 사용하는 애너테이션이다.

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적 메서드 전용
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{}
```

### 메타 애너테이션

애너테이션 선언에 다는 애너테이션을 말한다. 위 예제에서는 @Retention과 @Target이 메타 애너테이션에 해당한다.

- @Retention(RetentionPolicy.RUNTIME) - 보존 정책
    - @Test가 런타임에도 유지되어야 한다는 표시
- @Target(ElementType.METHOD) - 적용 대상
    - @Test가 반드시 메서드에 선언되어야 한다는 표시

이러한 마커 애너테이션은 적절한 애너테이션 처리기가 필요하다. 마커 애너테이션은 단순히 Test라는 이름에 오타가 있거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 내어주는 역할을 할 뿐이다. 실제 클래스가 동작하는데 직접적인 영향을 미치는 게 아니고 유용한 정보를 제공할 뿐이다.

```java
// 애너테이션 처리기
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
        // 수정된 부분
        if (m.isAnnotationPresent(ExceptionTest.class)) {
            tests++;
            try {
                m.invoke(null);
                System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
            } catch (InvocationTargetException wrappedEx) {
                Throwable exc = wrappedEx.getCause();
                Class<? extends Throwable> excType =
                        m.getAnnotation(ExceptionTest.class).value();
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
    }
    System.out.printf("성공: %d, 실패: %d%n",
            passed, tests - passed);
  }
}
```

- 리플렉션을 이용하여 마커 애너테이션을 찾고, 예외 발생 시 InvocationTargetException으로 Wrapping 된다. 그래서 해당 예외에 담긴 실패 정보를 추출해서 출력한다.
- 해당 애너테이션의 의도와 다르게 사용되어졌을 경우**(매개변수 없는 정적 메서드가 아닌 경우)**에는 InvocationTargetException 외의 다른 예외가 발생되는데, 이를 처리하기 위해서는 새로운 애너테이션 타입이 필요하다.

### 매개변수를 가진 애너테이션

```java
// 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

- 이 애너테이션의 매개변수 타입은 Class<? extends Throwable>이다.
- Throwable을 확장한 클래스의 Class 객체를 뜻한다. 즉, 모든 예외 타입을 수용한다는 뜻이다.

```java
public class Sample {
	// 성공해야한다.
	@ExceptionTest(ArithmeticException.class)
	public static void m1() {
		int i = 0;
		i = i / i;
	}

	// 실패해야한다. (다른 예외 발생)
	@ExceptionTest(ArithmeticException.class)
	public static void m2() {
		int[] a = new int[0];
		int i = a[1];
	}
}
```

- 위와 같이 발생할 예외를 인자로 넘겨주면 된다. 애너테이션 처리기에서는 다음과 같이 코드를 수정한다.

```java
// 애너테이션 처리기
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclaredMethods()) {
        // 수정된 부분
        if (m.isAnnotationPresent(ExceptionTest.class)) {
            tests++;
            try {
                m.invoke(null);
                System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
            } catch (InvocationTargetException wrappedEx) {
                Throwable exc = wrappedEx.getCause();
                Class<? extends Throwable> excType =
                        m.getAnnotation(ExceptionTest.class).value();
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
    }
    System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
  }
}
```

더 나아가 예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. 기존 애너테이션에 Class 객체를 배열로 수정해보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

그리고 애너테이션 처리기의 구현을 다음과 같이 변경한다.

```java
public class RunTests {
	  public static void main(String[] args) throws Exception {
      int tests = 0;
      int passed = 0;
      Class<?> testClass = Class.forName(args[0]);
      for (Method m : testClass.getDeclaredMethods()) {
          // 수정됨
          if (m.isAnnotationPresent(ExceptionTest.class)) {
            tests++;
            try {
              m.invoke(null);
              System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
            } catch (Throwable wrappedExc) {
              Throwable exc = wrappedExc.getCause();
              int oldPassed = passed;
              Class<? extends Throwable>[] excTypes =
                      m.getAnnotation(ExceptionTest.class).value();
              for (Class<? extends Throwable> excType : excTypes) {
                  if (excType.isInstance(exc)) {
                      passed++;
                      break;
                  }
              }
              if (passed == oldPassed)
                  System.out.printf("테스트 %s 실패: %s %n", m, exc);
            }
          }
      }
	  }
}
```

- 반복문을 추가하여 Throwable 배열을 검증하는 로직이 추가되었다.

```java
// m2의 테스트도 통과
public class Sample {
	// 성공해야한다.
	@ExceptionTest(ArithmeticException.class)
	public static void m1() {
		int i = 0;
		i = i / i;
	}

	// 성공해야한다. (다른 예외 발생)
	@ExceptionTest({ArithmeticException.class, ArrayIndexOutOfBoundsException.class})
	public static void m2() {
		int[] a = new int[0];
		int i = a[1];
	}
}
```

### 반복 가능한 애너테이션

Java8 부터는 단일 요소에 애너테이션을 반복적으로 달 수 있는 @Repeatable 메타 애너테이션을 제공한다. 따라서 배열 매개변수 대신 @Repeatable 메타 애너테이션을 달면 단일 요소에 반복적으로 적용할 수 있게 되었다.

@Repeatable 사용 시 주의점

- @Repeatable을 명시한 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의해야 한다. 그리고 @Repeatable에 해당 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
- 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
- 적절한 @Retention과 @Target을 명시해야 한다. 그렇지 않으면 컴파일되지 않는다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] value();
}
```

적용 방법

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { }
```

- @ExceptionTest를 여러 개 달면 하나만 달렸을 때와 구분하기 위해 컨테이너 애너테이션 타입이 적용된다.
- 애너테이션 처리기에서 이 둘을 구분하기 위해서는 isAnnotationPresent로 검사를 수행해야 한다. isAnnotationPresent는 이 둘을 명확히 구분할 수 있기 때문이다.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
    }
}
```

- 이러한 방법은 코드 가독성을 높일 수 있지만 애너테이션을 처리하는 부분의 코드가 늘어나고 복잡해서 오류가 발생할 확률이 커진다는 것을 명심하자.

### 핵심 정리

- **애너테이션으로 처리할 수 있다면 명명 패턴을 사용할 이유는 없다.**
- 어떤 도구 제작자를 제외하고는 애너테이션 타입을 직접 정의할 일은 거의 없다. 하지만 **자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용하자.**

# 아이템 40 @Override 애너테이션을 일관되게 사용하라

@Override 애너테이션은 상위 타입의 메서드를 재정의했음을 나타낸다. 메서드 선언에만 달 수 있으며, 일관되게 사용하면 여러 가지 버그들을 예방할 수 있다.

### @Override는 의도를 명확히 나타낸다.

@Override 애너테이션을 달아줌으로써 **'재정의한다.'라는** 의도를 명확히 나타낼 수 있다. 또한, 코드를 잘못 작성**(Overriding이 아닌 Overloading)**했을 경우 컴파일러가 잘못된 부분을 명확히 알려준다.

### @Override를 달지 않아도 되는 단 하나의 예외

구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 @Override를 달지 않아도 된다. 구체 클래스인데 아직 구현하지 않은 추상 메서드가 있다면 컴파일러가 이를 알려주기 때문이다.

하지만 @Override를 일괄적으로 다는 게 좋아 보인다면 @Override를 달아줘도 상관없다.

### @Override를 붙여주는게 좋은 이유

Java8에서 디폴트 메서드를 지원하기 시작하면서, 인터페이스의 추상 메서드를 구현한 메서드에도 @Override를 붙여주는 게 좋다. 이렇게 하면 메서드의 시그니처가 올바른지 재차 확신할 수 있기 때문이다.

또한, 실수로 추가한 메서드가 있는지 확인하기 위해 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 붙여주는 게 좋다.

그러니 상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자.

# 아이템 41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

### 마커 인터페이스

- 아무 메서드도 담고 있지 않고 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 **마커 인터페이스**라 한다.
- 대표적으로 Serializeable 인터페이스를 예로 들 수 있다. Serializeable은 자신을 구현한 클래스의 인스턴스는 직렬화할 수 있음을 알려주는 역할을 한다.
- 마커 애너테이션이 등장하면서 마커 인터페이스가 더 이상 쓸모없게 느껴질 수 있겠지만, 사실 마커 인터페이스는 두 가지 측면에서 마커 애너테이션보다 더 나은 점이 있다.

### 마커 애너테이션 VS 마커 인터페이스

마커 인터페이스가 가지는 강점

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 사용할 수 있다.
- **마커 애너테이션과**는 다르게 **마커 인터페이스**는 타입이기 때문에 마커 애너테이션을 사용했다면 런타임에야 발견될 오류를 컴파일 타임에 잡아낼 수 있다.
- 예를 들어 ObjectOutputStream,writeObject 메서드는 인수로 받은 객체가 Serializable을 구현했을 거라 가정한다. 그런데 이 메서드는 Object 객체를 받도록 설계되어 있다. 따라서 Serializable을 구현하지 않은, 즉 직렬화할 수 없는 객체를 넘기면 런타임에야 문제를 확인할 수 있게된다.
1. 적용 대상을 보다 정밀하게 지정할 수 있다.
- 적용 대상(@Target)을 타입으로 주었을 경우(ElementType.Type)에는 클래스, 인터페이스, 열거 타입, 애너테이션등 모든 타입에 애너테이션을 달 수 있다. 더 세밀하게 애너테이션을 달 수 없다는 뜻이다. 하지만 **마커 인터페이스는 특정 구현 클래스에만 적용하는 것이 가능하다.**

마커 애너테이션이 가지는 강점

1. 거대한 애너테이션 시스템의 지원을 받을 수 있다.
- 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 사용하는 쪽이 일관성을 지키는 데 유리하다.

사용 기준

그렇다면 언제 마커 애너테이션을 사용해야 하고, 또 어떤 때 마커 인터페이스를 사용해야 하는 걸까?

- **클래스와 인터페이스 외의 요소(모듈, 패키지, 필드, 지역변수)에 마킹을 해야 한다면 애너테이션을 사용할 수밖에 없다.** 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문이다.
- **마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용하자.**
    - 이렇게 하면 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일 타임에 오류를 잡아낼 수 있다. 이런 메서드를 작성하지 않는다는 확신이 있다면 마커 애너테이션이 더 나은 선택이 될 것이다.

핵심 정리

- 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 사용하자.
- 클래스나 인터페이스 외의 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크에 사용될 마커라면 애너테이션을 사용하자.

---

추가 학습 자료

https://jojoldu.tistory.com/231

https://www.inflearn.com/course/the-java-code-manipulation#
