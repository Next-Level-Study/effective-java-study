정적 메서드와 정적 필드만을 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설게한게 아니지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 
추상클래스로 만드는 것은 하위클래스를 만들어 인스턴스화 할수 있고 상속해서 사용할수 있다는 오해를 살수 있으므로 쓰면 안된다

private 생성자를 사용한다면 사용자는 생성자를 호출할수 없다. 또한 하위 클래스에서ㅓ 상위 클래스의 생성자에 접근이 불가능 함으로 상속을 불가능하게 하는 효과도 있다

private 생성자 내부에서 Assertion Error를 던지는 것은 클래스 안에서 생성자를 호출하는 것을 막아 실수로 생성자를 호출하지 못하도록 한다

추후 사용자의 편의를 위해 적절한 주석을 담아 놓는 것이 좋다
java.lang.Math 
```java
/**  
* Don't let anyone instantiate this class.  
*/  
private Math() {}
```

java.util.Collections
```java
// Suppresses default constructor, ensuring non-instantiability.  
private Collections() {  
}
```