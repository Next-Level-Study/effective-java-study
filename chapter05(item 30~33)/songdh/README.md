# Item 30 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로 메서드도 제네릭으로 만들 수 있다.
명시적으로 형변환 해야하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.
## 단순 제네릭 메서드 작성법
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```
1. 메서드 선언에서의 집합(입력값 s1, s2, 반환값)의 원소타입을 타입 매개변수로 명시하고 메서드 안에서도 타입 매개변수만을 사용하게 수정
2. 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다

#### 타입 매개변수의 명명규칙
제네릭 메서드나 제네릭 타입이나 같음
타입 매개변수 이름은 보통 한 문자로 표현한다. 대부분은 다음 중 하나다.

|       |                  |
| ----- | ---------------- |
| **T** | 임의의 타입      |
| **E** | 컬렉션 원소 타입 |
| **K** | 맵의 키          |
| **V** | 맵의 값          |
| **X** | 예외             |
| **R** | 메서드 반환 타입 |
| T,U,V / T1,T2,T3      | 임의의 타입 시퀀스                 |

이 방삭은 집합의 원소타입이 모두 같아야함
한정적 와일드 카드 타입으로 유연하게 개선 할 수 있음

## 제네릭 정적 싱글턴 팩터리
- 요청타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 정적팩터리를 만드는 패턴-
- 불변 객체를 여러타입으로 활용할수 있게 만들어야할 때는 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화할 수 있음
- 항등함수를 담은 클래스의 경우
	- 자바의 제네릭이 실체화된다면 항등함수를 타입별로 하나씩 만들어야 했겠지만 소거방식을 사용하여 제네릭 싱글턴 하나만 있으면됨
```java
//Function.identity();
static <T> Function<T, T> identity() {  
	return t -> t;  
}
//Collections.reverseOrder();
public static <T> Comparator<T> reverseOrder() {  
	return (Comparator<T>) ReverseComparator.REVERSE_ORDER;  
}
public static <T> Comparator<T> reverseOrder(Comparator<T> cmp) {  
	if (cmp == null) {  
		return (Comparator<T>) ReverseComparator.REVERSE_ORDER;  
	} else if (cmp == ReverseComparator.REVERSE_ORDER) {  
		return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;  
	} else if (cmp == Comparators.NaturalOrderComparator.INSTANCE) {  
		return (Comparator<T>) ReverseComparator.REVERSE_ORDER;  
	} else if (cmp instanceof ReverseComparator2) {  
		return ((ReverseComparator2<T>) cmp).cmp;  
	} else {  
		return new ReverseComparator2<>(cmp);  
	}  
}
```
### 재귀적 타입한정
- 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정 하는 방법
- 타입의 자연적 순서를 정하는 Comparable 인터에이스와 함께쓰임
	- 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있음
	- Comparable의 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교 될 수 있어야 한다
	- 이 제약을 코드로 표현한 것
``` java
		public static <E extends Comparable<E>> E max(Collection<E> c);
```
	 - 타입한정인 <E extends Comparable<E>>은 모든 타입 E는 자신과 비교할 수 있다라고 읽을 수 있다
	상호 비교 가능하다는 뜻
	- 재귀적 타입한정은 복잡해질 가능성이 있긴하지만 잘 일어나지 않음

# Item 31 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변
하지만 불공변 방식보다 유연한게 필요한 경우가 존재함

## 한정적 와일드카드타입
- 매개변수화 타입은 불공변이기에 하위타입이더라도 타입이 다르면 오류발생
- 이런경우에 한정적 와일드카드타입이라는 특별한 매개변수화 타입을 지원
- `Iterable<? extends E>`는 E의 하위타입의 Iterable 이라는 뜻
- 유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드타입을 사용하라
- 반환타입은 한정적 와일드 카드를 사용하면 안됨. 클라이언트 코드에서도 와일드카드 타입을 사용해야하기때문
- 와일드 카드게 바르게 사용됐다면 클래스 사용자는 와일드 타입이 쓰였다는 사실을 의식하지 못하고 사용하는게 가능
- 클래스 사용자가 와일드 카드 타입을 신경써야 한다면 API에 문제가 있을 가능성이 큼

> [!note] 와일드카드 타입 지정 공식
> 펙스(PECS) : producer-extends, consumer-super
> 매개변수화 타입 T가 생산자라면 `<? extends T>` 를 사용하고 소비자라면 `<? super T>`를 사용하라
> 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 타입을 명확히 지정해야하기 때문에 와일드 카드 타입을 쓰지 않는 게 좋다 

예시)
`public Chooser(Collection<T> choices)` -> `public Chooser(Collection<? extends T> choices)`
T타입의 값을 생산하고 나중을 위해 저장하는 케이스는  T를 확장하는 와일드카드 타입을 사용해 선언

#### 자바 7 이하에서 와일드 카드타입 사용시 주의할 점
- 자바 7에서는 타입 추론 능력이 강력하지 못해 문맥에 맞는 반환타입을 명시했어야하는 문제점이 존재
- 명시적 타입 인수를 사용하여 타입을 알려주면 해결됨
``` java
	Set<Number> numbers = Union.<Number>union(integers, doubles);
```

#### 타입매개변수에 와일드타입을 적용한 예시
``` java
public static <E extens Comparable<? super E>> E max(List<? extends E> list)
```
1. 입력 매개변수에서는 E 인스턴스를 생산하므로 `List<E>`를 `List<? extends E>`로 수정
2. 타입매개변수에는 E가 `Comparable<E>`를 확장한다고 정의했는데 이때 `Comparable<E>`는 E인스턴스를 소비하므로 매개변수화타입은 `Comparable<? super E>`를 사용함
	- 이때 `Comparable` 과 `Comparable`는 언제나 소비자임으로 `Comparable<? super E>`와 `Comparator<? super E>`를 사용하는게 낫다
##### 이렇게 수정했을 때의 이점
Comparable(혹은 Comparator)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원할 수 있음

## 타입매개변수와 와일드카드 중 선택
타입매개변수와 와일드카드에는 공통되는 부분이 있어서 메서드를 정의할 때 둘중 어느 것을 사용해도 괜찮을 때가 많다
```java
1. 비한정적 타입 매개변수를 사용
public static <E> void swap(List<E> list, int i, int j);
2. 비한정적 와일드카드를 사용
public static void swap(List<?> list, int i, int j);
```
public API라면 간단한 2번이 낫다
신경 써야할 타입 매개변수가 없고 어떤 타입이 들어와도 잘 동작을 할 것이기 때문에 사용자 입장에서 심플한 것을 제공
### 기본규칙
메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체
비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고 한정적타입 매개변수라면 한정적 와일드카드로 변경

# Item 32 제네릭과 가변인수를 함께 쓸 때는 신중하라

## 가변인수
- 가면인수는 메서드에 넘기는 인수의 개수를 클라인언트가 조절할 수 있게 해줌
#### 구현 방식의 허점
- 가변인수메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어져 내부로 감추어야 했을 배열을 클라이언트에노출하는 문제가 발생함
- 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생
- 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보냄
- 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면 그 호출에 대해서도 경고를 발생 시킴
- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생
- 컴파일러가 자동 생성한 형변환이 실패할 수 있어 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들림

> 안전하지 못함에도 제네릭 varages 매개변수를 받는 메서드를 선언 할 수 있게 한ㅊ이유
> 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 유용하기 때문
## @SafeVarargs
- 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 됨
- 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치
- 안전한게 확실하지 않다면 절대 달면 안됨

#### `@SafeVarargs`를 사용해야할 때
제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드
-> 안전하지 않으면 쓰지 말 것
-> 재정의 할수 없는 메서드에만 달아아한다
	- java8에서는 정적메서드와 final 인스턴스 메서드에서만 사용가능하다
	- java9에서는 private인스턴스 메서드에서도 허용
#### 메서드가 안전한지 확인 하는 방법
- 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 메서드는 안전
-  varargs 매개변수 배열에 아무것도 저장하지 않는 것
- 제네릭 varargs 매개변수 배열 혹은 복제본에 다른 메서드가 접근하도록 허용하면 안전하지 않다
	- 예외
		1. `@SafeVarargs`로 제대로 애노테이트 된 또다른 varargs 메서드에 넘기는것
		2. 이 배열 내용의 일부 함수를 호출만하여 varargs를 받지 않는 일반 메서드에 넘기는것
이미 생성된  `@SafeVarargs`가 달린 메서드를  활용하는 것도 한가지 방법
- 대신 코드가 조금 지저분해지고 속도가 느려질 수 있다

# Item 33 타입 안정 이종 컨테이너를 고려하라

제네릭은 단일 원소 컨테이너에서 흔히 쓰인다
이 쓰임에서 매개변수화 되는 대상은 원소가 아니고 컨테이너 자신이기에 하나의 컨테이너에서 매개변수화할수 있는 타입의 수가 제한된다
보통은 상관 없지만 유연한 수단이 필요할 때가 있다 

## 타입안전 이종  컨테이너 패턴
사용예 - db에서 행은 임의의 열을 가질 수 있는데 모든 열을 타입안전하게 이용할수 있게 하는 방법
- 컨테이너 대신 키를 매개변수화 한다음 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공
- 각 타입의 class 객체를 매개변수화한 키 역할로 사용

#### 타입 토큰
- 컴파일 타임 타입정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴

```java
public class Favorites {
	// 와일드카드타입이 중첩되어 모든 키가 서로다른 매개변수화 타입일 수 있다
	// 값의 타입이 Object 모든 값이 키로 명시한 타입임을 보증하지 않지만 이 관계가 성립됨을 알기때문에 사용가능함
	private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
				favorites.put(Objects.requireNonNull(type), instance);
		}	

	// 반환할 때는 타입링크 정보가 버려지기에 잘못된 컴파일 타임 타입을 가지고 있어 형변환을 해주어야함
	public <T> T getFavorite(Class<T> type) {
		// type.cast를 통해 T로 동적 형변환
		return type.cast(favorites.get(type));
	}
}

// class.cast();
public T cast(Object obj) {  
	if (obj != null && !isInstance(obj))  
		throw new ClassCastException(cannotCastMsg(obj));  
	return (T) obj;  
}
```

#### 제약
1. 악의적으로 로타입으로 넘기면 타입안전성이 깨짐
	1. 하지만 비검사 경고가 뜸으로 감수하겠다면 런타임시에는 안전성을 확보할 수있음
	2. 그리고 보장하기 위해서는 명시된 타입과 같은지를 확인하면됨
2. 실체화 불가 타입에서는 사용할 수 없음
	1. Class 객체를 얻을 수 없기때문` List<String>.class`등은 오류 발생
	2. List.class 리터럴에 제네릭 List 타입 객체를 모두 넣는다면 데이터가 섞여 문제가 발생할것
	3. 슈퍼타입 토큰으로 일정부분 완화 할 수도 있기는 하지만 한계점이 있으니 주의해서 사용해야한다

## 한정적 타입 토큰
- 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입 토큰
- 애너테이션 API에서 적극적으로 사용하고 있음
```java
public <U> Class<? extends U> asSubclass(Class<U> clazz) {  
	if (clazz.isAssignableFrom(this))  
		return (Class<? extends U>) this;  
	else  
		throw new ClassCastException(this.toString());  
}
```
- asSubclass() 메서드는 특정 클래스의 하위 타입의 한정적 타입 토큰으로 캐스팅해주는 역할
- `<? extends Annotation>`은 `<T extends Annotation>`에 호환 가능.
