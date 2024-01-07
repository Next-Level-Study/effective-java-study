# Chapter 7 람다와 스트림

### 자바 8

- 함수형 인터페이스
- 람다
- 메서드 참조
- 스트림 API: 데이터 원소의 시퀀스 처리 라이브러리 지원

# 아이템 42.  익명 클래스보다는 람다를 사용하라

### 함수 객체

- 예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다.
- 이런 인터페이스를 함수 객체(function object)라고 하여, 특정 함수나 동작을 나타내는 데 썼다.
- JDK 1.1에서 함수 객체를 만드는 주요 수단으로 익명 클래스를 사용

### 익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법이다!

```java
public static void main(String[] args) {
    List<String> words = new ArrayList<>();
    Collections.sort(words, new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            return Integer.compare(o1.length(), o2.length());
        }
    });
}
```

- 전략 패턴처럼, 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했다.
- 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.
- 자바 8에 와서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받았고, 지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 람다식(lambda expression)을 사용해 만들 수 있게 되었다.
    - 람다는 함수나 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결하다.

### 람다식을 함수 객체로 사용 - 익명 클래스 대체

```java
public static void main(String[] args) {
    List<String> words = new ArrayList<>();
    // 컴파일러 타입 추론
    Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
}
```

- 여기서 람다, 매개변수(s1, s2), 반환값의 타입은 각각 (`Comparator<String>`), String, int지만 코드에서는 언급이 없다. 이는 컴파일러가 문맥을 살펴 타입을 추론해준 것이다.
- 상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다.
- **타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하도록 한다.**
    - 컴파일러는 타입을 추론하는 데 필요한 타입 정보 대부분을 제네릭에서 얻는다.
    - 따라서 우리가 이 정보를 제공하지 않으면 컴파일러는 람다의 타입을 추론할 수 없게 되어, 결국 일일이 타입 정보를 명시해야 한다.

### 함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입

```java
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator operator;

    Operation(String symbol, DoubleBinaryOperator operator) {
        this.symbol = symbol;
        this.operator = operator;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y) {
        return operator.applyAsDouble(x, y);
    }
}
```

- 람다를 이용하면 열거 타입의 인스턴스 필드를 이용하는 방식으로 상수별로 다르게 동작하는 코드를 쉽게 구현할 수 있다.
- 단순히 각 열거 타입 상수의 동작을 람다로 구현해 생성자에 넘기고, 생성자는 이 람다를 인스턴스 필드로 저장해둔다.
- 그런 다음 apply 메서드에서 필드에 저장된 람다를 호출하기만 하면 된다.

### 람다를 사용할 때의 고려할 점

1. 메서드나 클래스와 달리, **람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.**
    - 람다는 한 줄일 때 가장 좋고 길어야 세 줄안에 끝내는 게 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠진다.
    - 만약 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩터링 하도록 한다.
2. 열거 타입 생성자에 넘겨지는 인수들의 타입도 **컴파일타임에 추론**된다. 따라서 열거 타입 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다(인스턴스는 런타임에 만들어지기 때문이다).
    - 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.
3. 람다는 함수형 인터페이스에서만 쓰인다.
    - 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다.
    - 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다.
4. 람다는 자신을 참조할 수 없다.
    - 람다에서의 this 키워드는 바깥 인스턴스를 가리킨다.
    - 반면 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다.
    - 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.
5. 람다도 익명 클래스처럼 직렬화 형태가 구현별로(가령 가상머신별로) 다를 수 있기 때문에, **람다를 직렬화하는 일은 극히 삼가야 한다(익명 클래스의 인스턴스로 마찬가지다).**
    - 만약 직렬화해야만 하는 함수 객체가 있다면(가령 Comparator처럼) private 정적 중첩 클래스의 인스턴스를 사용하도록 한다.

### 핵심 정리

- 자바 8에서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었으며, 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있다.
- **익명 클래스는 함수형 인터페이스가 아닌 타입의 인스턴스를 만들 때만 사용하도록 한다.**

# 아이템 43.  람다보다는 메서드 참조를 사용하라

- 람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함이다.
- 자바에는 함수 객체를 람다보다도 더 간결하게 만드는 방법이 있는데, 메서드 참조(method reference)다.

### 메서드 참조(Method reference)

- 자바 8이 되면서 Interger 클래스와 모든 기본 타입의 박싱 타입은 람다와 기능이 같은 정적 메서드들을 제공하기 시작했다.
- 람다 대신 이 메서드의 참조를 전달하면 똑같은 결과를 더 보기 좋게 얻을 수 있다.
    - 예) Integer::sum
- 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다.
- 그래도 메서드 참조를 사용하는 편이 보통은 더 짧고 간결하므로, 람다로 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이 될 수 있다.
    - 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 사용하는 식이다.
    - 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 설명을 문서로 남길 수도 있다.

### 메서드 참조의 유형

1. 정적 메서드 참조

- 가장 흔한 메서드 참조 유형으로 정적 메서드를 참조한다. 앞서 살펴본 Integer::sum도 정적 메서드에 속한다.

```java
// 메서드 참조
Integer::sum

// 람다
str -> Integer.pasrseint(str)
```

2. 한정적 인스턴스 메서드 참조

- 참조 대상 인스턴스를 특정하는 한정적 인스턴스 메서드 참조 유형이다. 근본적으로 정적 참조와 비슷하여 함수 객체가 받는 인수와 참조되는 메서드가 받은 인수가 똑같다.

```java
// 메서드 참조
Instant.now()::isAfter

// 람다
Instant then = Instant.now();
t -> then.isAfter(t);
```

한정적 인스턴스 메서드 참조는 인스턴스의 함수 결과 값의 메서드를 참조한다.

3. 비한정적 인스턴스 메서드 참조

- 비한정적 인스턴스 참조는 함수 객체를 적용하는 시점에 수신 객체를 알려준다. 이를 위해 수신 객체 전달용 매개변수가 목록의 첫 번째로 추가되며, 그 후로 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다.

```java
// 메서드 참조
String::toLowerCase

// 람다
str -> str.toLowerCase()
```

- 비한정적 인스턴스 메서드 참조는 주로 Stream 파이프라인에서의 매핑과 필터 함수에 쓰인다.

4. 클래스 생성자 참조

- 클래스의 생성자를 참조한다. 생성자 참조는 팩터리 객체로 사용된다.

```java
// 메서드 참조
TreeMap<K,V>::new

// 람다
() -> new TreeMap<K, V>()
```

5. 배열 생성자 참조

```java
// 메서드 참조
int[]::new

// 람다
len -> new int[len]
```

### 제네릭 함수 타입(generic function type)

```java
interface G1 {
    <E extends Exception> Object m() throws E;
}

interface G2 {
    <F extends Exception> String m() throws Exception;
}

interface G extends G1, G2 {
}
```

- 람다로는 불가능하지만 메서드 참조로는 가능한 유일한 예가 제네릭 함수 타입 구현이다.
- 함수형 인터페이스의 추상 메서드가 제네릭일 수 있듯이 함수 타입도 제네릭일 수 있다.
- 위 코드에서 함수형 인터페이스 G를 함수 타입으로 표현하면 `<F extends Exception> () -> String throws F` 와 같다.
- 함수형 인터페이스를 위한 제네릭 함수 타입은 메서드 참조 표현식으로는 구현할 수 있지만, 람다식으로는 불가능하다. 제네릭은 람다식이라는 문법이 존재하지 않기 때문이다.

### 핵심 정리

- 메서드 참조는 람다의 간단명료한 대안이 될 수 있다.
- **메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하도록 한다.**

# 아이템 44.  표준 함수형 인터페이스를 사용하라

- 자바가 람다를 지원하면서 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.
- 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것으로 대체할 수 있기 때문이다.
- → 함수 객체를 매개변수로 받는 생성자와 메서드를 생성해야 한다.
- 이때, 함수형 매개변수 타입을 올바르게 선택해야 한다.

### 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라.

```java
@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```

- 위 인터페이스는 잘 동작하지만, 굳이 사용할 이유는 없다. 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다.
- java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨 있다.
- **필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하도록 한다.**
    - API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워질 것이다.
    - 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 좋아질 것이다.

### 표준 함수형 인터페이스

java.util.function은 총 43개의 함수형 인터페이스를 제공한다. 그 중 다음 기본 인터페이스 6개만 기억하면 다른 인터페이스를 유추하기 쉬워진다.

1. UnaryOperator<T>
- T 타입 인수 하나를 받아서 T 타입을 반환하는 함수형 인터페이스

```java
public static void main(String[] args) {
	UnaryOperator<Integer> plus10 = (i) -> i + 10;
	UnaryOperator<Integer> multiply2 = (i) -> i * 2;

	plus10.andThen(multiply2).apply(2); // (10 + 2) * 2 = 24
	plus10.compose(multiply2).apply(2); // (2 * 2) + 10 = 14
}
```

1. BinaryOperator<T>
- T 타입 인수 두 개를 받아서 T 타입을 반환하는 함수형 인터페이스

```java
public static void main(String[] args) {
	BinaryOperator<Integer> plus = (a,b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

1. Predicate<T>
- T 타입을 받아서 boolean을 리턴하는 함수형 인터페이스

```java
public static void main(String[] args) {
	Predicate<String> isHello = (s) -> s.startWith("Hello");
	isHello.test("Hi"); // false
	isHello.test("Hello"); // true
}
```

1. Function<T, R>
- T 타입을 받아서 R 타입을 리턴하는 함수형 인터페이스다. 인수와 리턴 타입이 다르다.

```java
public static void main(String[] args) {
	Function<Integer, String> print10 = (s) -> String.valueOf(s);
	System.out.println(print10 .apply(10)); // "10" 출력
}
```

1. Supplier<T>
- T 타입의 값을 제공하는 함수형 인터페이스로 **공급자**라고도 불린다. 인수를 받지 않고 값을 반환한다.

```java
public static void main(String[] args) {
	Supplier<Integer> get10 = () -> 10; // 10을 리턴하겠다!
	get10.get();
}
```

1. Consumer<T>
- T 타입을 받아서 아무 값도 리턴하지 않는 함수형 인터페이스다. 소비자라고도 한다.

```java
public static void main(String[] args) {
	Consumer<Integer> printT = (i) -> System.out.println(i);
	printT.accept(10); // 10 출력
}
```

1. BiFunction<T, U, R>
- 두 개의 입력 값(T, U)를 받아서 R 타입을 리턴하는 함수형 인터페이스

```java
public static void main(String[] args) {
	BiFunction<Integer, Integer, Integer> plus = (a, b) -> a + b;
	System.out.println(plus.apply(10, 20)); // 30 출력
}
```

기본 인터페이스는 기본 타입인 int, double, long으로 각 3개씩 변형이 생긴다. 예를 들면, int를 받는 Predicate는 IntPredicate가 되고, long을 받는 BinaryOperator는 LongBinaryOperator가 되는 식이다.

더 많은 인터페이스를 확인하고 싶다면 **[java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)**을 참고하자.

### 언제 표준 함수형 인터페이스를 사용해야 할까?

표준 함수형 인터페이스

- 표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 **기본** 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 않도록 한다.
    - 계산량이 많을 때는 성능이 처참히 느려질 수 있다.
- 표준 함수형 인터페이스 중 필요한 용도에 맞는 게 없다면 직접 작성해야 하며, 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다.

### Comparator<T> 인터페이스 - 직접 작성하는 케이스

구조적으로 ToIntBiFunction<T, U>와 동일하지만 독자적인 인터페이스로 존재해야 하는 이유

- API에서 굉장히 자주 사용되는데, 이름이 그 용도를 아주 훌륭히 설명해준다.
- 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
- 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 많이 담고 있다.

Comparator의 특성

이 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 고민해보도록 해야 한다.

1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
2. 반드시 따라야 하는 규약이 있다.
3. 유용한 디폴트 메서드를 제공할 수 있다.

만약 전용 함수형 인터페이스를 작성하기로 했다면, '인터페이스'임을 명심해야 한다. 아주 주의해서 설계해야 한다.

### @FunctionalInterface 애너테이션

이 애너테이션을 사용하는 이유는 @Override를 사용하는 이유와 비슷하다. 프로그래머의 의도를 명시하는 것으로, 크게 세 가지 목적이 있다.

- 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
- 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.

따라서, **직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하도록 한다.**

### 함수형 인터페이스를 사용할 때의 주의점

- 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다.
    - 클라이언트에게 불필요한 모호함만 줄 뿐이며, 이 때문에 실제로 문제가 일어나기도 한다.

### 핵심 정리

- 자바 8부터 람다를 지원한다. 입력값과 반환값에 함수형 인터페이스 타입을 활용하도록 한다.
- 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다.
- 흔치 않지만, 가끔은 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있다.

# 아이템 45.  스트림은 주의해서 사용하라

### 스트림 API

- 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 위해 자바 8에 추가되었다.
- 스트림 API가 제공하는 추상 개념의 핵심
    - 스트림(stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 뜻한다.
    - 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.
    - 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다.
        - 기본 타입 값으로는 int, long, double 이렇게 세 가지를 지원한다.

### 스트림 파이프라인

- 스트림 파이프라인은 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝난다.
- 이 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.
    - 각 중간 연산은 스트림을 어떠한 방식으로 변환(transform)한다.
- 스트림 파이프라인은 지연 평가(lazy evaluation)된다.
    - 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.
    - 종단 연산이 없는 스트림 파이프라인은 아무 일도 하지 않는 명령어인 no-op과 같으므로, 종단 연산은 필수다.
- 스트림 API는 메서드 연쇄를 지원하는 플루언트 API(fluent API)다.
    - 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있으며, 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다.
- 기본적으로 스트림 파이프라인은 순차적으로 수행된다.
    - 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 된다. 다만, 효과를 볼 수 있는 상황은 많지 않다.

```java
// 사전을 하나를 훑어 원소 수가 많은 아나그램 그룹들을 출력한다.
public class Anagrams {
    public static void main(String[] args) {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner scanner = new Scanner(dictionary)) {
            while (scanner.hasNext()) {
                String word = scanner.next();
                groups.computeIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values()) {
            if (group.size() >= minGroupSize) {
                System.out.println(group.size() + ": " + group);
            }
        }
    }

    private static String alphabetize(String word) {
        char[] a = word.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- 자바 8에서 추가된 computeIfAbsent 메서드
    - 이 메서드는 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환한다.
    - 만약 키가 없으면 건네진 함수 객체를 키에 적용하여 값을 계산하고, 그 키와 값을 매핑한 뒤 계산된 값을 반환한다.

```java
// 스트림을 과하게 사용했다. - 따라 하지 말 것!
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        // 스트림을 과하게 사용했다. - 따라 하지 말 것!
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new, (sb, c) -> sb.append((char) c), StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }

    private static String alphabetize(String word) {
        char[] a = word.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- 코드는 확실히 짧지만 읽기 어렵다.
- 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

```java
// 스트림을 적절히 사용하면 깔끔하고 명료해진다.
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static String alphabetize(String word) {
        char[] a = word.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

- 람다 매개변수의 이름은 주의해서 정해야 한다.
- **람다에서는 타입 이름을 자주 생략하므로 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.**
- 도우미 메서드를 적절히 활용하는 일의 중요성은 일반 반복 코드에서보다 스트림 파이프라인에서 훨씬 크다. 파이프라인에서는 타입 정보가 명시되지 않거나 임시 변수를 자주 사용하기 때문이다.
- 자바는 기본 타입인 char용 스트림을 지원하지 않는다.
    - char은 int 값을 갖기 때문이고, 그 덕에 int 스트림을 반환하면 헷갈릴 수 있다. 올바르게 동작하게 하려면 명시적으로 형변환을 해줘야 한다.
    - 따라서 **char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.**
- 스트림이 언제나 가독성과 유지보수 측면으로 뛰어난 것은 아니다.
    - 스트림과 반복문을 적절히 조합하는 게 최선이다.
    - 따라서 **기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영하도록 한다.**

### 스트림이 안성맞춤인 일

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등).
- 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며).
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

### 스트림으로 처리하기 어려운 일

- 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하는 것은 처리하기 어렵다.
    - 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.

### 스트림과 반복 중 어느 쪽을 써야 할까?

1. 데카르트 곱 계산을 반복 방식으로 구현

```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values()) {
        for (Rank rank : Rank.values()) {
            result.add(new Card(suit, rank));
        }
    }
    return result;
}
```

1. 데카르트 곱 계산을 스트림 방식으로 구현

```java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit -> Stream.of(Rank.values())
                    .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
```

- 중간 연산으로 사용한 flatMap은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다.
- 이러한 작업을 평탄화(flattening)라고도 한다.

### 핵심 정리

- 스트림과 반복 방식은 각각에 알맞은 일이 있다.
- 수 많은 작업은 이 둘을 조합했을 때 가장 멋지게 해결된다.
- **만약 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 선택하도록 한다.**
