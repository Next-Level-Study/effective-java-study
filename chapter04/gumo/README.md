# Item 15. 클래스와 멤버의 접근 권한을 최소화하라.
> ✔ 잘 설계된 컴포넌트는 모든 내부 구현을 외부 컴포넌트로부터 완벽히 숨겨, 구현과 API를 깔끔하게 분리한다.

이는 `정보 은닉` 혹은 `캡슐화`로 이어진다. </br>
정보를 은닉함으로 `컴포넌트를 독립`시켜 개발, 테스트, 최적화, 적용, 분석, 수정을 개별적으로 할 수 있게 해준다.

### 👍 모든 클래스와 멤버의 접근성을 가능한 좁혀야 한다.
> ✔ 공개 API가 된다는 것은, 하위 호환을 위해 계속 관리해야 하고 문서를 제공해야할 수도 있다는 말을 포함하도 있다

1. 패키지 외부에서 쓸 이유가 없다면 package-private로 선언하자.

2. 한 클래스에서만 사용하는 package-private 톱레벨 클래스/인터페이스는 이를 사용하는 클래스에 private static으로 중첩시키자. </br>

3. public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. </br>
public으로 하면 <u>필드의 값을 제한할 힘을 잃게 된다.</u> </br>
가변 객체를 참조하는 경우에는, 클래스의 객체는 참조하지 못하지만, 참조된 객체 자체는 수정될 수 있으므로 접근자 메서드를 유의해서 작성하자.
    1. public 배열을 private으로 바꾸고, `Collections.unmodifiableList`로 불변 리스트를 추가한다.
    2. public 배열을 private으로 바꾸고, 반환하는 경우에는 그 복사본을 반환하자. (방어적 복사)

> ✔ 클라이언트가 무엇을 원하는지, 어느 반환 타입이 더 쓰기 편할지, 성능은 어느쪽이 나을지 고민하고 정하자.

이 외에, 상수의 경우에는 `public static final` 필드로 공개해도 된다. (UNMODIFIABLE_URL 이런식으로 명명하면 된다.)


### 멤버 접근성을 좁히지 못하게 하는 제약.
* 상위 클래스의 메서드를 재정의 하는 경우에는, 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다. </br>
상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 리스코프 치환 원칙을 지키기 위해 필요하다.

> ✔ 암묵적 접근 수준 : public, protected 지만, 모듈 내부로 한정되는 경우.

# Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.
```java
class Point {
    private double x;
    private double y;

    // constructor

    // getter, setter : 접근자와 변경자
}
```
접근자와 변경자를 제공하자. </br>
이렇게 하면 클래스 내부 표현 방식을 언제든 바꿀 수 있다.

# Item 17. 변경 가능성을 최소화하라.
> ✔ 불변 클래스 : 인스턴스 내부 값을 수정할 수 없는 클래스

### 불변 클래스로 만든다면?
1. 가변 클래스보다 설계하고 구현하고 사용하기 쉽다. 
2. 오류가 생길 여지가 적어 훨씬 안전함.
3. 불변 객체는 근본적으로 스레드 세이프하여 따로 동기화할 필요없다. 
4. 불변 객체에 대해 스레드 간에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다.

그래서 불변 클래스의 인스턴스는 최대한 재활용하기를 권한다. <br>
가장 쉬운 재활용 방법은 상수로 제공하는 것. `public static final`

하지만, 값이 다른 경우 반드시 독립된 객체로 만들어야 한다. </br>
ex) BigInteger에서는 백만비트 중 한 비트를 바꾸기 위해서는 새로운 객체를 만들어야 한다.

### 클래스를 불변으로 만들기 위한 다섯 가지 규칙
1. 객체의 상태를 변경하는 메서드(변경자, setter)를 제공하지 않는다

2. 클래스를 확장할 수 없도록 한다.
    * 클래스를 final로 선언.
    * 생성자를 private/default로 만들고 public 정적 팩터리 제공.

3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
5. 자신 외에 내부의 가변 컴포넌트에 접근할 수 없도록 한다.


주로 자주 사용되는 인스턴스를 캐싱해서 같은 인스턴스를 중복 생성하기 않게 해주는 정적 팩터리를 제공할 수 있다.
```java
// Integer.java
@HotSpotIntrinsicCandidate
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }


private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

# Item 18. 상속보다는 컴포지션을 사용하라.
상속은 캡슐화를 깨뜨린다.
상위 클래스가 내부 구현이 달라지면, 하위 클래스에 영향이 간다.

HashSet을 상속받아서 재정의하는 경우, 내부적으로 addAll이 add를 불러 생각 외로 작동한다.

상위 클래스의 메서드 동작을 다시 구현하는 방식은 어렵고, 시간도 들고, 자칫 오류를 내거나 성능을 떨어뜨릴 수도 있다

하위 클래스에서 반환 타입을 다르게 한다면, 컴파일조차 되지 않고 반환 타입이 같아지면 결국 재정의한 꼴이 된다.

그래서 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자. </br>
기존 클래스의 구성 요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션(구성)이라고 한다.

1. 기존 클래스의 내부 구현 방식의 영향에서 벗어난다.
2. 기존 클래스에 새로운 메서드가 추가되더라도 영향을 받지 않는다.
3. 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 하는 콜백 프레임워크와는 어울리지 않는다.

아래는 스택오버플로우에서 가져온 예제. <br>
내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 자신의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출한다.
```java
interface SomethingWithCallback {
    void doSomething();
    void call();
}


class WrappedObject implements SomethingWithCallback {
    // constructor

    @Override
    public void doSomething() {
        // ...
    }

    @Override
    public void call() {
        System.out.println("WrappedObject callback!");
    }
}


class Wrapper implements SomethingWithCallback {

    private final WrappedObject wrappedObject;

    // constructor

    @Override
    public void doSomething() {
        wrappedObject.doSomething();
    }

    @Override
    public void call() {
        System.out.println("Wrapper callback!");
    }
}

public static void main(String[] args) {
    WrappedObject wrappedObject = new WrappedObject(service);
    Wrapper       wrapper       = new Wrapper(wrappedObject);
    wrapper.doSomething();  // 이 경우에 wrappedObject의 doSomthing이 호출됨.
}   
```
상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야 한다.

# Item 19. 상속을 고려해 설계하고 문서화 하라. 그러지 않았다면 상속을 금지하라.
* 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다.
</br> 재정의 가능 메서드에서 재정의 가능 메서드를 호출하는 경우, 문서로 남겨라
* 상속용 클래스는 하위 클래스 3개정도를 만들어 시험해라.
* 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다. </br>
 상위 클래스 생성자가 호출되고, 하위 클래스의 재정의된 메서드를 호출할 때, 하위 클래스 생성자에서 초기화하는 값에 의존한다면 프로그램이 오동작할 것 이다.
 * `Cloneable`, `Serializable` 인터페이스는 상속용 설계에 어려움을 더한다. <br>
    * `clone`, `readObject`는 생성자와 같은 제약을 받는다.
    * `readResolve`, `writeReplace` 메서드를 갖는다면, 이 메서드는 하위 클래스에서 무시되는 private이 아닌 protected로 선언해야한다. 

> 상속용으로 설계하지 않은 클래스는 상속을 금지해라.

# Item 20. 추상 클래스보다는 인터페이스를 우선하라.
자바에서 다중 구현 메커니즘은 `인터페이스`와 `추상 클래스`로 제공된다.
* 기존 클래스에 추상 클래스를 끼워넣는건 어렵다.
* 인터페이스는 믹스인(클래스가 구현할 수 있는 타입) 정의에 맞춤이다. 이를 통해, 주된 타입 외에 선택적 행위를 제공한다고 선언하는 효과를 줄 수 있다.

### 추상 골격 구현 클래스
* 인터페이스와 추상 클래스의 장점을 모두 취하는 방법이다.
* ex. 컬렉션 프레임워크의 AbstractCollection, Abstractmap ...
* 인터페이스로 타입을 정의하고, 필요하다면 디폴트 메서드를 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다.
* 혹은 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 `시뮬레이트한 다중 상속`이라고 한다.
