# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라.
인터페이스에 메서드를 추가하면 구현체에 그 메서드가 존재할 가능성은 아주 낮기 때문에, 기존 구현체를 깨드리지 않고 인터페이스에 메서드를 추가할 방법은 없었다.</br>
Java8부터는 인터페이스에 디폴트 메서드를 추가할 수 있다.
```java
interface Test {
    void say();

    default void saySomething() {
        // 이 메서드를 구현하지 않는 클래스에서는, 여기서 구현된 내용이 사용된다.
    }
}
```

> 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다. </br> 하지만, 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는 데 유용한 수단이 된다.

왜 피해야 하는가? </br>
자바 8부터 Collection 인터페이스에 추가된 디폴트 메서드인 removeIf를 보자.

```java
public interface Collection<E> extends Iterable<E> {
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
}
```
위 메소드가 추가된 이후, Collection을 구현하는 아파치 커먼즈의 SynchronizedCollection은, removeIf를 구현하지 않아서 Collection 인터페이스의 removeIf 구현을 물려받게 되었다.

```java
public class SynchronizedCollection<E> extends Object implements Collection<E>, Serializable {

}
```

동기화를 제공하는 SynchronizedCollection이어야 하지만, removeIf를 구현하지 않게 되어, </br>멀티 스레드 환경에서는 removeIf를 호출하면 ConcurrentModificationException이 발생하는 예기치 못한 결과로 이어졌다.

* 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다.
* 인터페이스를 설계할 때는 주의를 기울여야 한다.

> 🤔 인터페이스를 저마다의 목적으로 구현하는 구현체들이 있을텐데, 구현체의 목적을 벗어나는 상황 혹은 오동작을 막기 위해서는 인터페이스를 구현하는 쪽에서 생각하고 신중하게 설계해야하는 것 같습니다. 

# Item 22. 인터페이스를 타입을 정의하는 용도로만 사용하라.
> 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.

### 인터페이스로 무엇을 할 수 있는가.
* 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 <strong>타입 역할</strong>을 할 수 있다.
* 인터페이스의 인스턴스가 <strong>무엇을 할 수 있는지</strong>를 클라이언트에게 알릴 수 있다.

### 인터페이스로 무엇을 하면 안되는가.
* 상수 인터페이스

#### 왜?
1. 클래스 내부에서 사용하는 상수는 내부 구현이 되어야 한다. 그러므로 상수 인터페이스를 사용하는 것은, 내부 구현을 클래스의 API로 노출하는 행위다.
2. 심하게는 클라이언트 코드가 내부 구현에 해당하는 상수들에 종속되게 한다.
3. final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 그 인터페이스가 정의한 상수들로 오염되어 버린다.

> 2번에 대해서는, 상수 인터페이스를 구현하는 클래스가 있다면 그 인스턴스를 통해 인터페이스의 상수를 외부 코드에서 사용할 수 있어서 그런 것 같음.
```java
public interface MathConstants {
    double PI = 3.141592;
}

public class Calculator implements MathConstants{}

public class Main {
    public static void main(String[] args){
        Calculator calculator = new Calculator();
        double pi = calculator.PI;
    }
}
```

### 상수를 공개한다면, 더 나은 방법들
* 클래스나 인터페이스 자체에 추가해라. ex)Integer나 Double의 MIN_VALUE, MAX_VALUE
```java
public final class Integer extends Number implements Comparable<Integer> {
    @Native public static final int   MIN_VALUE = 0x80000000;

    @Native public static final int   MAX_VALUE = 0x7fffffff;
}
```
* 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 공개해라.
* 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개해라.

# Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라.
#### 태그 달린 클래스?
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드
    final Shape shape;

    // 사각형/원용 생성자
    
    double area() {
        switch(shape) {
            case RECTANGEL:
                // ...
            case CIRCLE:
                // ...
            default:
                // ...
        }
    }
}
```

### 태그 달린 클래스는 단점이 한가득
1. 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많다.
2. 가독성 나쁘다
3. 필드를 final로 선언하려고 하면, 특정 태그에서 사용하지 않는 필드까지 초기화해야한다.
4. <strong>인스턴스 타입만으로는 의미를 알 수 없다.</strong>

> 자바같은 객체 지향 언어는, 타입 하나로 다양한 의미의 객체를 클래스 계층 구조로 표현하는 방법인 서브타이핑 방법이 있다.

```java
// 1. 계층구조의 루트가 될 추상 클래스를 정의한다.
abstract class Figure {
    //2. 태그 값에 따라 동작이 달라지는 메서드는 루트 클래스의 추상 메서드로 선언한다. 
    abstract double area();

    // 동작이 일정한 메서드는 루트 클래스의 일반 메서드로 추가한다.
}

class Circle extends Figure {
    // 클래스의 의미에 해당하는 final 데이터 필드만 갖게된다.
    final double radius;

    // constructor

    // 의미에 맞게 메서드를 구현하면된다.
    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}
```

* 타입 사이의 자연스러운 계층 관계를 반영할 수 있어, 유연성이 좋고 컴파일타임 타입 검사 능력을 높여준다는 장점이 있다.

# Item 24. 멤버 클래스는 되도록 static으로 만들라.
#### 중첩 클래스(nested class)
다른 클래스 안에 정의된 클래스이며, 자신을 감싼 바깥 클래스에서만 쓰여야 한다.
`정적 멤버 클래스`, `(비정적)멤버 클래스`, `익명 클래스`, `지역 클래스`의 종류가 있다.
첫번째를 제외한 나머지는 내부 클래스(inner class)에 해당한다.

#### 1. 정적 멤버 클래스
다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에 접근할 수 있다는 점 제외하고는 일반 클래스와 같다.</br>
다른 정적 멤버와 똑같은 규칙을 적용받는다.
```java
public class Calculator {

    public static class Embedded {
        public static void doSomething() {
            System.out.println("embedded");
        }
    }
}

public class Main {
    public static void main(String[] args){
        Calculator.Embedded.doSomething();
    }
}
```

#### 2. 비정적 멤버 클래스
비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결되어, 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.</br>
비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.
```java
public class Calculator {

    private final int number;
    private final Embedded embedded = new Embedded();

    // constructor

    public void doSomething() {
        embedded.doSomething();
    }

    private class Embedded {
        public void doSomething() {
            // 정규화된 this를 사용해서 바깥 인스턴스의 메서드를 호출할 수 있다.
            System.out.println(Calculator.this.number);
        }
    }
}
```
* (?) 관계 정보가 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다.
* 어답터를 정의할 때 자주 쓰인다.

> 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면, 무조건 static 을 붙여 정적 멤버 클래스로 만들자.

* static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 되고, 이 참조를 저장하려면 시간과 공간이 소비된다. 심각한 문제는, GC가 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다.
> Map 구현체는 엔트리 객체들을 가지고 있음. 엔트리를 비정적 멤버 클래스로 표현하는 것은 동작은 하겠지만 모든 엔트리가 바깥 맵으로의 참조를 갖게 되어 공간과 시간을 낭비할 것이다.

#### 3. 익명 클래스
* 바깥 클래스의 멤버가 아니다. 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
* 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
```java
// 부모 클래스
class Animal {
    public String bark() {
        return "동물이 웁니다";
    }
}

public class Main {
    public static void main(String[] args) {
        // 익명 클래스 : 클래스 정의와 객체화를 동시에. 일회성으로 사용
        Animal dog = new Animal() {
        	@Override
            public String bark() {
                return "개가 짖습니다";
            }
        };
        	
        // 익명 클래스 객체 사용
        dog.bark();
    }
}
// 새로 정의한 메소드는 외부에서 사용할 수 없다.
출처: https://inpa.tistory.com/entry/JAVA-☕-익명-클래스Anonymous-Class-사용법-마스터하기 [Inpa Dev 👨‍💻:티스토리]
```

#### 4. 지역 클래스
* 이름이 있고, 반복해서 사용할 수 있다.
* 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있고, 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 한다.
```java
public class Calculator {

    class Embedded {}
}
```

# Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라.
한 파일에 톱 클래스 파일이 여러개라면, 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라진다.

1. 톱레벨 클래스들을 서로 다른 소스 파일로 분리해라.
2. 그게 아니면, 정적 멤버 클래스를 사용해라.

