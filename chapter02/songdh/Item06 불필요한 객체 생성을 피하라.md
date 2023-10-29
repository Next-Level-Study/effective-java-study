같은 기능의 객체를 새로 생성하는 대신, 객체 하나를 재사용하는 편이 나을 때가 많다.
특히, 불변 객체는 언제든 재사용할 수 있다.

#### String
`String s = new String("java");`

String은 불변객체로 String을 new로 생성하면 항상 새로운 객체를 만들기에  아래와 같이 사용하는게 좋다

`String s = "java";`
이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 재사용 한다.

#### 정적 팩토리 메소드
생성자대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 불필요한 객체 생성을 피할 수 있다.
생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않다.
ex) Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메서드 사용

##### 생산비용이 비싼 객체
정규표현식용 Pattern은 인스턴스 생산비용이 비싼 객체인데 메소드를 부를 때마다 만드는 것보다 캐싱하고 쓰는 편이 좋다
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
static final로 선언하여 클래서 초기화 과정에 캐싱해두고 매소드가 호출될때마다 재사용 한다.

지연초기화도 있지만 코드가 복잡해지는 반면 성능개선에 큰역할을 못할때가 있어 권하지는 않는다고 한다.

#### 오토박싱
오토박싱은 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술
```java
private static long sum() {
	Long sum = 0L;
	for(long i=0; i<=Integer.MAX_VALUE; i++) {
		sum += i;
	}
	return sum;
}
```

long 타입인 i가 Long 타입인 sum 인스턴스에 더해질 때마다 새로운 객체가 생성되어 느리다
박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

이 아이템으로 인해 "객체 생성은 비싸니 피해야 한다"로 오해하면 안 된다.
DB 커넥션같이 특별한 경우를 제외하고는 객체를 생성하고 회수하는 일이 크게 부담되지 않고 일반적으로는 프로그램의 명확성과 간결성, 기능을 위해서는 객체를 추가로 생성하는 일은 좋은 일이다.