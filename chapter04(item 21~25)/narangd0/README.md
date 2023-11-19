## 아이템 21 인터페이스는 구현하는 쪽을 생각해 설계하라

기존 구현체를 수정하지 않고 인터페이스에 메서드를 추가하는 방법

- 자바 8 - 기존 인터페이스에 메서드를 추가할 수 있는 디폴트 메서드 소개
- 자바 8의 Collection 인터페이스에 추가된 디폴트 메서드 예제

```jsx
default boolean removeIf(Predicate<? Super E> filter) {
	Objects.requireNonNull(filter);
	boolean result = false;
	for (Interator<E> it = iterator(); it.hasNext(); ) {
		if (filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}
	return result;
}	
```

- org.apache.commons.collections4.collection.SynchronizedCollection
    - (모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬랙션 객체에 기능을 위임하는 래퍼 클래스)
    - 이 클래스와 removeIf 메서드의 충돌을 예시로 인터페이스 디폴트 메서드 사용시 나타날 수 있는 문제를 설명함

기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피해야 한다.

- 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.
- 새로운 인터페이스라면 릴리즈 전에 반드시 테스트를 거치자.

## 아이템 22 인터페이스는 타입을 정의하는 용도로만 사용하라

> 인터페이스는 타입을 정의하는 용도로만 쓰자. 상수 공개용 수단으로 사용하지 말자.
> 

인터페이스

- 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할
- 인터페이스를 구현한 클래스는 자신의 인스턴스로 무엇을 할 수 있는지 용도를 클라이언트에게 얘기해주는 것

👎 상수 인터페이스

- 메서드 없이 static final 필드만 정의한 인터페이스

```jsx
public interface PhysicalConstants {
	// 아보가드로 수 (1/몰)
	static final double AVOGADROS_NUMBER = 6.02_1440_857e23;

	// 볼츠만 상수 (J/K)
	static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```

- 클래스 내부에서 사용하는 상수는 내부 구현에 해당하는 것인데, 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위이다.

상수를 공개할 목적이라면

- 특정 클래스, 인터페이스와 연관된 상수라면 그 클래스, 인터페이스 자체에 추가해야 한다.
    - 예) Integer, Double 에 선언된 MIN_VALUE, MAX_VALUE 상수
- 열거 타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개한다. (아이템 34)
- 인스턴스화 할 수 없는 유틸리티 클래스(아이템 4)에 담아 공개한다.

```jsx
public class PhysicalConstants {
	private PhysicalConstants() { } // 인스턴스화 방지

	// 아보가드로 수 (1/몰)
	public static final double AVOGADROS_NUMBER = 6.02_1440_857e23;

	// 볼츠만 상수 (J/K)
	public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
}
```

- 숫자 리터럴에 사용한 밑줄(_): 자바 7부터 허용, 숫자 리터럴의 값에는 아무런 영향을 주지 않으면서 읽기는 훨씬 편하게 해준다. 밑줄을 사용해 세 자릿씩 묶어주는 게 좋다.

## 아이템 23 태그 달린 클래스보다는 클래스 계층구조를 활용하라

태그(tag)

두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 나타내는 변수

👎 태그 달린 클래스

```jsx
class Figure {
  enum Shape {RECTANGLE, CIRCLE};

  // 태그 필드 - 현재 모양을 나타낸다.
  final Shape shape;

  // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
  double length;
  double width;

  // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
  double radius;

  // 원용 생성자
  public FigureWithTag(double radius) {
    this.shape = Shape.CIRCLE;
    this.radius = radius;
  }

  // 사각형용 생성자
  public FigureWithTag(double length, double width) {
    this.shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }

  double area() {
    switch (shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError(shape);
    }
  }
}
```

- 단점
    - enum 타입 선언, tag 필드(shpae), switch 문 등 쓸데 없는 코드가 많다.
    - 여러 구현이 한 클래스에 혼합되어 있어 가독성이 나쁘다.
    - 다른 의미를 위한 코드도 함께 하기 때문에 메모리도 많이 사용한다.
    - 필드를 final로 선언하려면, 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다.
    - 다른 의미를 추가하려면 코드를 수정해야 한다.
    
    → 장황하고, 오류를 내기 쉽고, 비효율적이다.
    

클래스 계층구조를 활용하는 서브타이핑(subtyping)

- 태그 달린 클래스를 클래스 계층구조로 바꾸는 방법
    1. 계층구조의 루트(root)가 되는 추상 클래스를 정의한다.
    2. 태그 값(의미)에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
    3. 태그 값에 상관없이 동작이 일정한 메서드들은 루트 클래스에 일반 메서드로 추가한다.
    4. 공통으로 사용하는 데이터 필드들도 전부 루트 클래스에 정의한다.
    5. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.
    6. 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드를 넣는다.

클래스 계층구조

```jsx
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {

    private final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {

    private final double width;
    private final double heigth;

    Rectangle(double width, double heigth) {
        this.width = width;
        this.heigth = heigth;
    }

    @Override
    double area() {
        return width * heigth;
    }
}
```

- 장점
    - 간결하고 명확하며, 쓸데없는 코드도 사라졌다.
    - 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다.
    - 살아남은 field들은 모두 final
    - 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해 준다.
    - 루트 클래스의 코드를 건드리지 않고도 독립적으로 계층 구조를 확장하고 함께 사용할 수 있다.
    - 타입이 의미별로 따로 존재하니 변수의 의미를 명시하거나 제한할 수 있고, 또 특정 의미만 매개변수로 받을 수 있다.
    - 자연스러운 계층 관계를 반영할 수 있다.
        
        ```jsx
        // 정사각형 클래스를 지원하도록 수정
        class Square extends Rectangle {
          Square(double width, double heigth) {
            super(width, heigth);
          }
        }
        ```
        

## 아이템 24 멤버 클래스는 되도록 static으로 만들라

중첩클래스

- 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
- 종류
    - 정적 멤버 클래스
    - 내부 클래스(inner class)
        - 멤버 클래스
        - 익명 클래스
        - 지역 클래스

정적 멤버 클래스

- 바깥 클래스의 정적 멤버와 똑같은 접근 규칙을 적용받는다.
- 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다.
    - 예) 계산기가 지원하는 언산 종류를 정의하는 Operation enum type
    
    ```jsx
    public class Calculator {
        public static enum Operator {
            PLUS("+", (x, y) -> x + y),
            MINUS("-", (x, y) -> x - y);
        }
    }
    ```
    

비정적 멤버 클래스

- 비정적 멤버 클래스의 인스턴스는 암묵적으로 바깥 클래스의 인스턴스와 연결된다.
- 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 참조를 가져올 수 있다.
    - 정규화된 this: *클래스명*.this 형태로 바깥 클래스의 이름을 명시하는 용법
- 따라서 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다. 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.
- ❓ 어댑터를 정의할 때 자주 쓰인다.
    - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것.

멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.

- static을 생략하면(멤버 클래스로 생성하면) 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.
- 이 참조를 저장하려면 시간과 공간이 소비되며, 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못해 메모리 누수가 발생할 수 있다.

private 정적 멤버 클래스

- 바깥 클래스가 표현하는 객체의 한 부분을 나타낼 때 쓴다.
- 예) Map 구현체가 갖는 키-값 쌍을 표현하는 엔트리(Entry) 객체
    - 모든 엔트리가 맵과 연관되어 있지만, 엔트리의 메서드들은 맵을 직접 사용하지 않는다.
        - 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비

익명 클래스

- 이름이 없다.
    - instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 여러 인터페이스 구현이나 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 짧지 않으면 가독성이 떨어진다.
- 정적 팩토리 메서드 구현

```jsx
static List<Integer> intArrayAsList(int[] a) {
	Objects.requireNonNull(a);

	return new AbstractList<>() {  // 익명 클래스 쓰임
		@Override
		public Integer get(int i) {
			return a[i];
		}

		@Override
		public Integer set(int i, Integer val) {
			int oldVal = a[i];
			a[i] = val;
			return oldVal;
		}
		...
	};
}
```

지역 클래스

- 지역변수를 선언할 수 있으면 어디든 선언 가능, 유효 범위도 지역변수와 같다.
- 멤버 클래스처럼 이름이 있고 반복해서 사용 가능
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성해야 한다.

정리

- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적(static)으로 만들자.
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.

## 아이템 25 톱레벨 클래스는 한 파일에 하나만 담으라

> 소스 파일을 어떤 순서로 컴파일하든, 바이너리 파일이나 프로그램 동작이 달라지는 일은 없어야 한다.
> 

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하는 건 아무런 득도 없고 위험을 감수해야 한다.

- 문제점
    - 한 클래스를 여러 가지로 정의할 수 있게 되고
    - 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일 하느냐에 따라 달라진다.

```jsx
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

```jsx
// 두 클래스를 한 파일(Utensil.java)에 정의
class Utensil {
    static final String NAME = "pan";
}

class Dessert{
    static final String NAME = "cake";
}
```

- Main을 실행하면 pancake 출력

```jsx
// 우연히 똑같은 두 클래스를 한 파일(Dessert.java)에 정의
class Utensil {
    static final String NAME = "pot";
}
class Dessert{
    static final String NAME = "pie";
}
```

1. javac Main.java Dessert.java 컴파일
    - (운 좋게) 컴파일러가 클래스가 중복 정의되었다고 알려준다.
2. javac Main.java 또는 javac Main.java Utensil.java 컴파일
    - pancake 출력
3. javac Dessert.java Main.java 명령으로 컴파일
    - potpie 출력
- 이처럼 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 바로잡아야 한다.

해결 방법

- 톱레벨 클래스 Utensil과 Dessert를 서로 다른 소스 파일로 분리한다.
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스(아이템 24)로 만든다.
    - 읽기 좋고, private으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문
    
    ```jsx
    // 톱레벨 클래스들을 정적 멤버 클래스로 바꾼 것
    public class Test {
        public static void main(String[] args) {
            System.out.println(Utensil.NAME + Dessert.NAME);
        }
    
        private static class Utensil {
            static final String NAME = "pan";
        }
    
        private static class Dessert{
            static final String NAME = "cake";
        }
    }
    ```
