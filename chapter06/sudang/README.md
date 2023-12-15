> 자바에는 특수한 목적의 참조 타입이 두 가지가 있다. 클래스의 일종인 열거 타임(enum), 인터페이스의 일종인 애너테이션이다. 이 타입들을 올바르게 사용하는 방법에 대해 알아보자.
>

---

## Item 34 int 상수 대신 열거 타입을 사용하라

### 열거 타입 사용 전

- 정수 열거 패턴 (int enum pattern)

    ```java
    public static final int APPLE_FUJI = 0;
    public static final int APPLE_PIPPIN = 1;
    public static final int APPLE_GRANNY_SMITH = 2;
    
    public static final int ORANGE_NAVEL = 0;
    public static final int ORANGE_TEMPLE = 1;
    public static final int ORANGE_BLOOD = 2;
    ```

    - 단점
        1. 타입 안전 보장 불가능, 표현력 좋지 않음
        2. APPLE_과 ORANGE_를 비교(==)를 비교해도 컴파일러의 경고는 없음
        3. 별도 이름공간(namespace)을 지원하지 않기 때문에 접두어를 써서 충돌을 방지 (ELEMENT_MERCURY, PLANET_MERCURY)
        4. 상수 값이 변경되면 클라이언트도 반드시 다시 컴파일해야 됨
- 문자열 열거 패턴 (string enum pattern)
    - 단점
        1. 상수의 의미를 출력할 수 있으나 하드코딩하게 만들 수 있음
        2. 하드코딩된 문자열 값에 오타가 있어도 컴파일러가 잡지 못해 런타임 오류 발생
        3. 문자열 비교로 인한 성능 저하

### 열거 타입

- 완전한 형태의 클래스 == 상수 하나당 인스턴스를 하나씩 만들어 public static final 필드로 공개
    - 생성자를 공개하지 않기 때문에 인스턴스 하나인 싱글턴 형태 ⇒ *인스턴스 통제*
- 장점
    - 컴파일 타입 안정성 제공
    - 각자의 이름공간이 있어 이름이 같은 상수도 공존 가능 == 상수 추가, 순서 교체를 해도 다시 컴파일하지 않아도 됨
    - 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현할 수 있음
        - 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장 (열거 타입은 근본적으로 불변이라 모든 필드가 *final*)
    - 열거 타입에서 상수 하나 제거해도 제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없음
        - 참조하던 클라이언트가 있어도 컴파일 시 오류를 발생시켜 가장 바람직한 대응을 함

- values() - 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드 (선언된 순서로 저장)
- valueOf(String) - 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환
    - toString() 메서드를 재정의하려거든, toString()이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString() 메서드도 함께 제공하는 걸 고려해보자
- 선언한 클래스 혹 그 패키지에서만 유용한 기능은 private나 package-private 메서드로 구현
- 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만듦

- 상수별 메서드 구현 (constant-specific method implementation)
    - 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body), 즉 각 상수에서 자신에 맞게 재정의하는 방법

        ```java
        public enum Operation {
        		PLUS {public double apply(double x, double y){return x + y;}};
        		// ...
        
        		public abstract double apply(double x, double y);
        }
        ```

        - 재정의하지 않았다면 컴파일 오류로 알려줌

- 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요함

    ```java
    private static final Map<String, Operation> stringToEnum 
    		= Stream.of(vlaues()).collect(toMap(Object::toString, e -> e));
    ```

    - 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없음
    - 추가적으로, 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없음
- 열거 타입에 새로운 값을 추가해 그 값을 처리하도록 처리해야되는 switch 문을 사용하는 메서드가 있다면 ‘전략’을 선택하도록 해보자
    - 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있음
        - 추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용
- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없음

---

## Item 35 ordinal 메서드 대신 인스턴스 필드를 사용하라

- ordinal() - 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메서드
    - 잘못된 예

        ```java
        public enum Ensemble {
        		SOLO, DUET, TRIO, QUARTET, QUINTET;
        
        		public int numberOfMusicians() { return ordinal() + 1; }
        }
        ```

        - 유지보수하기 끔찍하고, 순서를 바꾸는 순간 오동작하며, 사용중인 정수와 같은 값의 상수는 추가할 수 없음
        - 해결책 : 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드로 저장하자

            ```java
            public enum Ensemble {
            		SOLO(1), DUET(2), TRIO(3) ... OCTET(8), DOUBLE_QUARTET(8);
            
            		private final int numberOfMusicians;
            		Ensemble(int size) { this.numberOfMusicians = size; }
            		public int numberOfMusicians() { return ordinal(); }
            }
            ```


---

## Item 36 비트 필드 대신 EnumSet을 사용하라

### 비트 필드

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }
}
```

- 비트 필드 (bit field) - 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모아 만들어진 집합

    ```java
    text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    ```

    - 비트 필드를 사용하면 비트별 연산을 이용해 합집합, 교집합 같은 집합 연산을 효율적으로 수행할 수 있음
    - 단점
        1. 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어려움
        2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다로움
        3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측해 적절한 타입(32bit or 64bit)를 선택해야 함

### java.util 패키지의 EnumSet

- 열거 타입 상수 값으로 구성된 집합을 효과적으로 표현해주며, Set 인터페이스를 완벽히 구현, 타입 안전, 다른 Set과도 함께 사용 가능함
- 내부는 비트 벡터로 구현되어있어, 원소가 총 64개 이하라면 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여줌

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) { ... }
}
```

- 집합 생성 등 다양한 기능의 정적 팩터리를 제공 - ex) of

    ```java
    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    ```


---

## Item 37 ordinal 인덱싱 대신 EnumMap을 사용하라

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[])new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden)
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

- 문제
    1. 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않음
    2. 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 함
    3. 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 운이 좋으면 예외를 던짐

### 해결책

- EnumMap - 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체
    - 내부에서 배열을 사용하기 때문에 EnumMap의 성능이 ordinal을 쓴 배열과 비견됨 (Map의 타입 안정성과 배열의 성능을 모두 얻어냄)
    - 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공함
    - 스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있음
        1. EnumMap 사용하지 않는 방법

            ```java
            Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle));
            ```

            - EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있음
        2. EnumMap을 이용해 데이터와 열거타입을 매핑하는 방법

            ```java
            Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet()));
            ```

            - 맵을 빈번히 사용하는 프로그램에서는 꼭 필요할 것
        - EnumMap 버전은 언제나 식물의 생명주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생명주기에 속하는 식물이 있을 때만 만들게 됨

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

- 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없음
- 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것
- 전이 하나를 얻으려면 이전 상태와 이후 상태가 필요하니 맵 2개를 중첩하면 쉽게 해결할 수 있음

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
    
            private static final Map<Phase, Map<Phase, Transition>> m 
    							= Stream.of(values()).collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class), toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));
    		    
    				public static Transition from(Phase from, Phase to) {
    				     return m.get(from).get(to);
    				}
    		}
    }
    ```

    - (x,y) → y는 선언만 하고 실제로 쓰이지 않는데, 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리(telescoping factory)를 제공하기 때문
    - 잘못 수정될 가능성이 적고, 내부가 배열로 구현되어 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 쉬움

---

## Item 38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 열거 타입은 확장할 수 없고, 대부분의 상황에선 좋지 않은 상황
    - 확장할 수 있는 열거 타입이 어울리는 쓰임 == 연산 코드 (operation code or opcode)
- 인터페이스를 사횽해 확장을 구현해 낼 수 있음
    - 사소한 문제
        - 열거 타입끼리 구현을 상속할 수 없음
    - 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법 존재
    - 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것

- 방법 1 : <T extends Enum<T> & Operation>

  ⇒ Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻

- 방법 2 : Collection<? extends Operation>

  ⇒ 유연하지만 특정 연산에서 EnumSet, EnumMap을 사용하지 못함


---

## Item 39 명명 패턴보다 애너테이션을 사용하라

### 명명 패턴

- 단점
    1. 오타가 나면 안됨
    2. 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없음
    3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없음

### 마커 애너테이션

```java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME) // 런타임 유지
@Target(ElementType.METHOD) // 메서드 선언에서만 사용
public @interface Test {}
```

- 메타 애너테이션 : 애너테이션 선언에 다는 애너테이션 (ex) @Retention, @Target
- 메서드 주석의 제약을 컴파일러가 강제하기 위해서는 적절한 애너테이션 처리기를 직접 구현해야 함
    - javax.annotation.processing API 문서 참고
- 마커 애너테이션 : 아무 매개변수 없이 단순히 대상에 마킹
    - 관심 있는 프로그램에게 추가 정보를 제공

```java
/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

- Class<? extends Throwable> == Throwable을 확장한 클래스의 Class 객체
    - 모든 예외와 오류 타입을 다 수용함

        ```java
        @ExceptionTest(ArithmeticException.class)
        public static void m1() { // 성공
        				int i = 0;
        				i = i / i;
        }
        
        @ExceptionTest(ArithmeticException.class)
        public static void m2() { // 실패 (다른 예외 발생)
        				int[] a = new int[0];
        				int i = a[1];
        }
        
        @ExceptionTest(ArithmeticException.class)
        public static void m3() {} // 실패 (예외가 발생하지 않음)
        ```


- @Repeatable 메타 애너테이션
    - 주의할 점
        1. @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 함
        2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 함
        3. 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용대상(@Target)을 명시해야 함

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
    ```

    - 반복 가능 애너테이션을 여러개 달아도 컨테이너 애너테이션 덕분에 명확히 구분됨

        ```java
        @ExceptionTest(Exception1.class)
        @ExceptionTest(Exception2.class)
        public static void doublyBad() { ... }
        ```

    - 반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높일 수 있지만 이를 선언하고 처리하는 부분에서는 코드 양이 늘어나고 복잡해져 오류가 날 가능성이 커짐을 명심하자

---

## Item 40 @Override 애너테이션을 일관되게 사용하라

- 메서드 선언에만 달 수 이쓰며 상위 타입의 메서드를 재정의했음을 뜻함
- 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 됨
- 재정의할 의도였으나 실수로 새로운 메서드를 추가했을 때 알려주는 컴파일 오류의 보완재 역할

---

## Item 41 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

- 마커 인터페이스 : 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스
- 마커 인터페이스가 마커 애너테이션보다 나은 두 가지 면
    1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않음
        - 마커 인터페이스는 어엿한 타입으로 런타임에야 발견될 오류를 컴파일 타입에 잡을 수 있음
    2. 적용 대상을 더 정밀하게 지정할 수 있음
        - 부착할 수 있는 타입을 @Target보다 더 세밀하게 제한하지 못함
        - Set 인터페이스도 일종의 제약이 있는 마커 인터페이스로 볼 수 있음
- 마커 인터페이스는 객체의 특정 부분을 불변식으로 규정하거나, 그 타입의 인스턴스는 다른 클래스의 특정 메서드가 처리할 수 있다는 사실을 명시하는 용도로 사용할 수 있을 것
- 반대로, 마커 애너테이션이 마커 인터페이스보다 나은 점으로는 거대한 애너테이션 시스템의 지원을 받는다는 점을 들 수 있음

- 클래스와 인터페이스 외의 프로그램 요소 - 애너테이션
- 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있으면 - 인터페이스
- 애너테이션을 활발히 활용하는 프레임워크에서 사용하려는 마커라면 - 애너테이션