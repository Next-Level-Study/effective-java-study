### 제네릭

제네릭을 사용하면 컬렉션이 담을 수 있는 타입을 컴파일러에게 알려주게 된다. 이에 컴파일러는 형변환 코드를 추가함으로써 런타임에서의 형변환 오류를 방지하고 컴파일 과정에서 오류를 차단하여 안전하고 명확한 프로그램을 만들 수 있다.

제네릭의 이점은 최대로, 단점은 최소화하는 방법을 알아본다.

# 아이템 26 로 타입은 사용하지 말라

### ❓제네릭 클래스와 제네릭 인터페이스

- 클래스, 인터페이스 선언에 타입 매개변수를 사용한 것
- 제네릭 타입이라고 한다.

```java
interface List<E> { }
```

### 로 타입

- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 것을 말한다.
- List<E>의 로 타입은 List
- 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작한다.

```java
private final Collection stamps = ...;

// 아무 오류 없이 컴파일 된다.
stamps.add(new Coint(...));

// 컬렉션에서 동전을 꺼내기 전까지 오류를 알아채지 못한다.
for (Iterator i = stamps.iterator(); i.hasNext()) {
	Stamp stamp = (Stamp) i.next();  // ClassCastException!
	stamp.cancel();
}
```

- 로 타입이 동작하는 이유
    - 제네릭 없이 짠 코드를 수용하기 위해 (제네릭은 자바 5부터 쓸 수 있다.)
    - 마이그레이션 호환성을 위해 로 타입을 지원하고 제네릭 구현에는 소거 방식(아이템 28)을 사용한다.

```java
List<String> stringTypeList = ...

public void rawTypeParamMethod(List rawType) {..}
public void objectTypeParamMethod(List<Object> objectType) {..}

rawTypeParamMethod(stringTypeList); // 가능
objectTypeParamMethod(stringTypeList);  // 불가능

// -> List<String> 은 List 의 하위 타입이나, List<Object> 의 하위 타입은 아니다
// -> 이는 List 와 같은 로 타입을 사용하면 타입 안전성을 잃게 됨을 의미한다.
```

### 비한정적 와일드카드 타입(unbounded wildcard type)

제네릭 타입을 쓰고싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때, 물음표(?)를 사용한다.

- Set<E> 의 비한정적 와일드카드 타입 → Set<?>
- 어떤 타입이라도 담을 수 있는 범용적인 매개변수화 타입이 된다.

### 이런 로 타입을 써도 되는 상황도 있다!

1. class 리터럴에는 로 타입을 써야 한다.
    - 자바 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다. (배열과 기본 타입은 허용한다)
        - List.class, String[].class, int.class  ⭕
        - List<String>.class, List<?>.class  ❌
2. 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정적 와일드 카드 타입 이외의 매개변수화 타입에는 적용할 수 없다.
    - ? 타입이든 로 타입이든 instanceof 연산자는 동일하게 동작한다.

# 아이템 27 비검사 경고를 제거하라

### 비검사 경고(Unchecked Warning)

- 주로 제네릭 타입을 사용할 때 발생, 컴파일러가 코드에서 타입 안전성을 보장할 수 없는 경우 발생하는 경고 메시지
- 런타임 시에 ClassCastException과 같은 예외가 발생할 수 있다는 것을 알려주는 것
- 종류
    - 비검사 형변환 경고
    - 비검사 메서드 호출 경고
    - 비검사 매개변수화 가변인수 타입 경고
    - 비검사 변환 경고 등

```java
Set<Lark> exaltation = new HashSet();

// javac 명령줄 인수에 -Xlint:uncheck 옵션을 추가하여 비검사 경고를 확인하자
Venery.java4: warning: [unchecked] unchecked conversion
				Set<Lark> exlatation = new HashSet();
															 ^
		required: Set<Lark>
		found: HashSet

// ---
// 자바 7부터 지원하는 다이아몬트 연산자(<>)를 통해 컴파일러가 알려준 타입 매개변수를 명시하지 않고 해결할 수 있다.
Set<Lark> exaltation = new HashSet<>();
```

### 할 수 있는 한 모든 비검사 경고를 제거하자.

타입 안정성이 보장되는 코드를 만들 수 있다. (런타임에 ClassCastException이 발생할 일이 없고, 의도한 대로 잘 동작하리라 확신할 수 있다.)

### 경고를 제거할 수 없지만 타입 안전하다고 확신할 수 있다면 @Suppress Warnings(”unchecked”) 에너테이션을 달아 경고를 숨기자.

안전하다고 검증되노 비검사 경고를 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.

**@SuppressWarnings 애너테이션**

- 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.
- 하지만 항상 가능한 한 좁은 범위에 적용하자.
    - 예) 아주 짧은 메서드, 생성자 등.
    - 절대 클래스 전체에 적용하지 않도록 한다.
- 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		// 생성한 배열과 배개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
		@SuppressWarnings("unchecked)
		T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
		return result;
	}
	...
}
```

### 정리

비검사 경고 제거 방법

1. 코드를 수정하여 타입 안전성을 보장
2. **`@SuppressWarnings("unchecked")`** 어노테이션을 사용하여 해당 경고를 무시
    - 어노테이션을 사용할 때에는 경고를 무시해도 안전한지 신중하게 판단
    - 무시해도 안전한 이유를 주석으로 남기기

# 아이템 28 배열보다는 리스트를 사용하라

### 배열

1. 공변(convariant): Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다. 즉, 함께 변한다.

```java
// 런타임에 실패한다.
Object[] objectArr = new Long[1];
objectArr[0] = "타입이 달라 넣을 수 없다.";  // ArrayStoreException
```

1. 실체화(reify): 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.

### 제네릭

1. 불공변(incovariant): 서로 다른 타입 Type1, Type2가 있을 때, List<Type1>은 List<Type2>의 하위 또는 상위 타입이 아니다.

```java
// 컴파일되지 않는다.
List<Object> objectList = new ArrayList<Long>();  // 호환되지 않는 타입이다.
objectList.add("타입이 달라 넣을 수 없다.");
```

1. 소거 방식: 타입 정보가 런타임에는 소거된다.
    - 원소 타입을 컴파일 타임에만 검사하며, 런타임에는 알 수 없다.
    - E, List<E>, List<String> - 실체화 불가 타입(non-reifiable type)
        - 실체화되지 않아 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다.
    - 실체화될 수 있는 타입: List<?>, Map<?,?> 같은 비한정적 와일드카드 타입
    

### 배열인 E[] 대신 컬렉션인 List<E>를 사용하자

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우에는 배열 대신 컬렉션을 사용하면 해결된다. 코드가 조금 복잡해지고 성능이 살짝 나빠질 수 있지만, 그 대신 타입 안전성과 상호운용성은 좋아진다.

```java
// 제네릭 적용 필요
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }

		// 1. 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다.
		// 2. 혹시 타입이 다른 원소가 들어 있다면, 런타임에 형변환 오류가 발생한다.
    public Object choose() {
        ThreadLocalRandom random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```

**Chooser를 제네릭으로 만들기 1 - 컴파일되지 않는다.**

```java
public class Chooser {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        ThreadLocalRandom random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```

- 위 클래스를 컴파일하면 오류 메시지가 출력될 것이다.
- 이를 해결하려면 Obejct[] 배열을 T 배열로 형변환하면 된다.

**Chooser를 제네릭으로 만들기 2 - 비검사 형변환 경고 발생**

```java
public class Chooser {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = (T[]) choices.toArray();
    }

    // ...
}
```

- T가 무슨 타입인지 알 수 없으니 컴파일러는 이 형변환이 런타임에도 안전한지 보장할 수 없다.
- 프로그램 자체는 동작하지만, 단지 컴파일러가 안전을 보장하지 못할 뿐이다.
- 만약 코드를 작성하는 사람이 안전하다고 확신한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 된다. 그래도 역시 가능한 한 경고의 원인을 제거하는 것이 좋다.

**리스트 기반 Chooser - 타입 안전성 확보**

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        ThreadLocalRandom random = ThreadLocalRandom.current();
        return choiceList.get(random.nextInt(choiceList.size()));
    }
}
```

# 아이템 29 이왕이면 제네릭 타입으로 만들라

제네릭 타입을 새로 만드는 방법을 알아보자

> Object 기반 스택 - 제네릭이 절실한 강력 후보!
> 

```java
public class StackBasedObject {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public StackBasedObject() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

일반 클래스를 제네릭 클래스로 만드는 방법

- 클래스 선언에 타입 매개변수를 추가한다.
    - 스택이 담을 원소 하나만 추가. 이때 타입 이름으로는 보통 E를 사용한다.

> 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다.
> 

```java
public class StackGeneric<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public StackGeneric() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러 발생
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

### 실체화 불가 타입으로는 배열을 만들 수 없다.

해결 방법

1. 제네릭 배열 생성을 금지하는 제약을 우회하는 방법으로, Object 배열을 생성한 다음 제네릭 배열로 형변환한다. 이렇게 하면 일반적으로 타입 안전하지 않지만 컴파일러는 오류 대신 경고를 내보낸다.
2. elements 필드의 타입을 E[]에서 Object[]로 바꾼다.

> 배열을 사용한 코드를 제네릭으로 만드는 방법 1
> 

```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다.
@SuppressWarnings("unchecked")
public StackGeneric() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

- 컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없다. 따라서 직접 타입 안전성을 해치지 않음을 확인하고 명시해야 한다.
- 비검사 형변환이 안전함을 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨기도록 한다.

> 배열을 사용한 코드를 제네릭으로 만드는 방법 2
> 

```java
public E pop() {
	  if (size == 0) {
	      throw new EmptyStackException();
	  }
	
		// push에서 E 타입만 허용하므로 이 형변환은 안전하다.
	  @SuppressWarnings("unchecked") 
	  E result = (E) elements[--size];
	
	  elements[size] = null; // 다 쓴 참조 해제
	  return result;
}
```

- E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 따라서 직접 증명하고 경고를 숨길 수 있다.

### 두 방식의 차이점

- 첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다. 그리고 코드도 더 짧다.
- 첫 번째 방식에서는 형변환을 배열 생성 시 단 한 번만 해주면 된다.
    - 하지만 E가 Object가 아닌 한 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution)을 일으키니 주의해야 한다.
- 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다.
