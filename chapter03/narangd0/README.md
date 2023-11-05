# Chapter 3.모든 객체의 공통 메서드

### 3장에서 다루게될 내용

- Object 클래스에서 final 이 아닌 메서드인 equals, hashCode, toString, clone, finalize
    - 모두 재정의(overriding)을 염두에 두고 설계된 것
- Comparable.compareTo

## Item 10. equals는 일반 규약을 지켜 재정의 하라

다음과 같은 상황에서는 재정의 하지 않는다.

1. 각 인스턴스가 본질적으로 고유하다.
    - 값이 아닌 동작을 표현하는 클래스. 예) Thread
2. 인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없다.
    - java.util.regex.Pattern: equals를 재정의해서 두 Pattern 인스턴스가 같은 정규표현식을 나타내는지 검사한다. → 논리적 동치성 검사
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
    
    ```jsx
    @Override public boolean equals(Object o) {
    	throw new AssertionError();  // 호출 금지
    }
    ```
    

### 논리적 동치성을 확인해야 하는데, 상위 클래스가 equals가 재정의 되지 않았을 때, 재정의 한다.

- 객체가 같은지가 아니라 값이 같은지 알고 싶을 때
    - 주로 Integer와 String처럼 값을 표현하는 클래스가 해당된다.
- 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하면 재정의 하지 않는다.
    - 인스턴스 통제 클래스(Item 1), Enum 클래스

### equals 메서드는 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.

- 반사성: x.equals(x) is true
- 대칭성: x.equals(y) is true, then y.equals(x) is true
- 추이성: x.equals(y) is true, and y.equals(z), then x.equals(z) is true
    - Point와 ColorPoint 예제. p.56
    - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
    - → 상속 대신 컴포지션을 사용하라! (Item 18) p.60
- 일관성: x.equals(y) is always true or false
    - java.net.URL의 equals:  호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 한다. equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안 된다!
    - equals는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.
- null 아님: x.equals(null) is false
    - instanceof 연산자를 활욘하여 입력 매개변수가 올바른 타입인지 검사하도록 한다.
    - instanceof: 첫 번째 피연산자가 null 이면 false를 반환한다.

### equals 메서드 구현 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 성능 최적화를 위해 자기 자신이면 true를 반환한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. ‘핵심’ 필드들이 모두 일치하는지 확인한다.
    - 기본 타입 필드: == 연산자로 비교
    - float, double 필드: Float.compare(float, float), Double.compare(double, double)로 비교
    → Float.equals, Double.equals 메서드는 오토박싱을 수반할 수 있어 성능상 좋지 않다.
    - 참조 타입 필드: 각각의 equals 메서드로 비교
    - 배열 필드: 원소 각각을 상기 지침대로 비교, 배열의 모든 원소가 핵심 필드라면 Arrays.equals 메서드를 사용

어떤 필드를 먼저 비교하냐에 따라 equals 성능을 좌지우지 한다. 다를 가능성이 더 크거나, 비교 비용이 싼 필드를 먼저 비교하자.

```jsx
@Override public boolean equals(Object o) {
	if (o == this)
		return true;
	if (!(o instanceof PhoneNumber))
		return false;
	PhoneNumber pn = (PhoneNumber) o;
	return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode = areaCode;
}
```

- equals를 재정의할 땐 hashCode도 반드시 재정의하자(Item 11)
- @Override 애노테이션을 사용하여 Objects.equals를 재정의함을 나타내자!
- 구글의 AutoValue 프레임워크를 사용하면 테스트 메서드를 알아서 작성해준다!

## Item 11. equals를 재정의하려거든 hashCode도 재정의하라

hashCode 일반 규약을 어기게 되면 해당 클래스의 인스턴스를 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생한다.

### 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

> equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
> 

```jsx
// 전형적인 hashCode 메서드
@Override public int hashCode() {
	int result = Short.hashCode(areaCode);
	result = 31 * result + Short.hashCode(prefix);
	result = 31 * result + Short.hashCode(lineNum);
	return result;
}
```

- 곱할 숫자, 31의 의미: 홀수이면서 소수(prime)인 값
    - 31 곱셈을 시프트 연산과 뺄셈으로 대체해 최적화할 수 있다.
    - 31 * i 는 (i << 5) - i 와 같다.

```jsx
// Objects 클래스의 hash 메서드 - 성능에 민감하지 않은 상황에서만 사용!
@Override public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

- 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하지 않고 캐싱하는 방식을 고려하자
- 단 이 타입의 객체가 주로 해시의 키로 사용될 것 같다면, 인스턴스가 만들어질 때 해시코드가 계산되어야 한다.
- 해시의 키로 사용되지 않는 경우라면, hashCode가 처음 불릴 때 계산하는 지연 초기화 전략을 써보자.
    - 필드를 지연 초기화하려면 그 클래스를 스레드 세이프하게 만들어야 한다.

```jsx
// 해시코드 지연 초기화 - 스레드 안전성 고려 필요!
private int hashCode;  // 자동으로 0으로 초기화 됨

@Override public int hashCode() {
	int result = hashCode;
	if (result == 0) {
		result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		hashCode = result;
	}
	return result;
}
```

- [참고] lombok - @EqualsAndHashCode
    - equals, hashCode 자동 생성
        - **equals** : 두 객체의 내용이 같은지, 동등성(equality) 를 비교하는 연산자
        - **hashCode** : 두 객체가 같은 객체인지, 동일성(identity) 를 비교하는 연산자
    - 자바 bean에서 동등성 비교를 위해 equals와 hashcode 메소드를 오버라이딩해서 사용하는데, @**EqualsAndHashCode**어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있다.
    - callSuper 속성
        - eqauls와 hashCode 메소드 자동 생성 시 부모 클래스의 필드까지 감안할지의 여부를 설정할 수 있다.
        - @EqualsAndHashCode(callSuper = true)로 설정 시 부모 클래스 필드 값들도 동일한지 체크하며, false(기본값)일 경우 자신 클래스의 필드 값만 고려한다.
    

## Item 12. toString을 항상 재정의하라

### toString 메서드

- Object의 기본 toString: **클래스_이름@16진수로_표시한_해시코드**를 반환한다.
- 객체를 println, prinft, 문자열 연결 연산자(+), assert 구문에 넘길 때, 디버거가 객체를 출력할 때 자동으로 불린다.
- toString은 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
- 모든 하위 클래스에서 이 메서드를 재정의 해야 한다.

- API 문서에 포맷을 명시하자
- toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자

```jsx
  /**
   * 전화번호의 문자열 표현을 반환한다.
   * xxx-yyy-zzzz 와 같은 형식으로 반환된다.
   * 만일, 각 자리 숫자의 갯수보다 적은 숫자를 넣게 되면 그 공간은 0으로 채워진다.
   */
  @Override
  public String toString() {
      return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
  }
```

## Item 13. clone 재정의는 주의해서 진행하라

### Cloneable 인터페이스

- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환한다.
- 만약 Cloneable 을 구현하지 않은 특정 인스턴스에서 clone 메서드를 호출하면 CloneNotSupportedException을 던진다.

### Object clone 메서드의 일반 규약

- x.clone() != x 는 참이다. 원본 객체와 복사 객체는 서로 다른 객체이다.
- x.clone().getClass() == x.getClass() 는 참이다. 하지만 반드시 만족해야 하는 것은 아니다.
- x.clone().equals(x) 는 참이지만 필수는 아니다.
- x.clone().getClass() == x.getClass(), super.clone()을 호출해 얻은 객체를 clone 메소드가 반환한다면, 이 식은 참이다.
- 관례상, 반환된 객체와 원본 객체는 독립적이어야 한다. 이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

### 필드가 기본 타입이거나 불변 객체인 경우

```jsx
@Override public PhoneNumber clone() {
	try {
  	return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
  	throw new AssertionError();
  }
}
```

- clone 메서드를 통해 클래스에 정의된 모든 필드도 원본 필드와 똑같은 값을 갖는다.
- 만약 모든 필드가 기본 타입이거나 불변 객체인 경우라면 더이상 손볼 것도 없다.

### 필드가 가변 객체인 경우: 재귀적인 clone 호출

```jsx
public class Stack implements Cloneable {
  private Object[] elements; // 가변 객체
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    System.out.println("---------------");
    System.out.println("Stack constructor");
    this.elements = new Foo[DEFAULT_INITIAL_CAPACITY];
  }

  @Override public Stack clone() {
    try {
      return (Stack) super.clone();
    } catch (CloneNotSupportedException e) {
      throw new AssertionError();
    }
  }
}
```

- 단순히 clone 메서드가 super.clone()을 호출한다면 elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조할 것이다.
- 결국 원본이나 복제본 중 하나를 수정하면 다른 하나도 수정된다. (shallow copy)

### 필드가 가변 객체인 경우: deepCopy를 할 때 반복적으로 복사하기

```jsx
@Override public Stack clone() {
	try {
    Stack stack = (Stack) super.clone();
    stack.elements = new Object[elements.length];
    for (int i = 0; i< elements.length; i++) {
    	stack.elements[i] = new Object();
    }
    return stack;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

### 복사 생성자와 복사팩토리 !!

## Item 14. Comparable을 구현할지 고려하라

### compareTo 메서드의 일반 규약

- compareTo는 기준 객체와 주어진 객체의 순서를 비교한다.
- **compareTo 반환값의 기준**
    - 기준 객체 < 주어진 객체 → -1 반환
    - 기준 객체 == 주어진 객체 → 0 반환
    - 기준 객체 > 주어진 객체 → 1 반환

### compareTo 작성 요령

- compareTo와 equals의 차이점
    1. Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 컴파일 타임에 인수 타입이 결정된다. 따라서 equals에서 했던 타입 확인이나 형 변환이 필요 없다.
    2. null을 인자로 받는다면 NullPointerException을 발생시키면 된다.
- compareTo 메서드는 equals와 마찬가지로 객체 참조 필드를 비교할 때 compareTo 메서드를 재귀 호출해서 순서를 비교한다.

### 값의 차를 기준으로 비교하는 비교자 (추이성 위배)

```jsx
static Compartor<Object> hashCodeOrder = new Comparator<>() {
	publicintcompare(Object o1, Object o2){
		return o1.hashCode() - o2.hashCode();
  }
}
```

이 방식은 사용하면 안 된다. 정수 오버플로우를 일으키거나 부동 소수점 계산 방식에 따른 오류를 낼 수 있다.

### 값의 차를 기준으로 비교하는 비교자 (대안)

- 정적 compare 메서드를 활용한 비교자

```jsx
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	publicintcompare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2,hashCode());
	}
};
```

- 비교자 생성 메서드를 활용한 비교자
