### 싱글턴 (singleton)
인스턴스를 오직 하나만 생성할 수 있는 클래스

싱글턴을 만드는 방식은 보통 두가지이다.
두방식 모두 생성자는 private로 감추고 인스턴스에 접근할 수단으로 public static 멤버를 마련해둔다

#### 1. public static 멤버가 final 필드인 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding()
}
```
private 생성자는 public static final필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.
public이나 protected 생성자가 없으므로 Elvis클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

##### 장점
해당 클래스가 싱글턴임이 API에 명백히 드러난다.
public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
간결하다.

리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있으므로 방어하기위해 두번째 객체가 생긴다면 예외를 던지도록 수정하면 된다.

#### 2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding()
}
```
Elvis.getInstance()는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis인스턴스는 만들어지지 않는다.
리플렉션을 통한 예외는 똑같이 적용된다.

##### 장점
API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다
원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다
정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다

싱글턴 클래스를 직렬화하려면 단순히 Serializable을 구현하고 선언하는 것만으로 부족하다.
모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야 한다.

이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

#### 3. 원소가 하나인 열거 타입을 선언하는 방식
대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

```java
public enum Elvis {
	INSTANCE; 

	public void leaveTheBuilding() { ... }
}

```

복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아주기 때문에 최선의 방법이다.
단, 만들려는 싱글턴이 Enum 이외의 다른 상위 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

