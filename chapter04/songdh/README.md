# Item15 클래스와 멤버의 접근 권한을 최소화하라

>잘 설계된 컴포넌트의 특징은 클래스의 내부 데이터와 내부 구현 정보를 완벽히 숨겨 구현과 api를 깔끔히 분리한다
>api를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다. => 정보 은닉 캡슐화 
### 정보 은닉
시스템을 구성하는 컴포넌트를 서로 독립시켜 개발, 테스트, 최적화, 적용, 분석, 수정을 개별적으로 할 수 있게 해 주는 것과 연관이 있음
- 시스템 개발 속도를 높인다 
- 시스템 관리 비용을 낮춘다
- 성능 최적화에 도움을 준다
- 소프트웨어 재사용성을 높인다
- 큰 시스템을 제작하는 난이도를 낮춰준다

> 모든 클래스와 멤버의 접근성을 가능한 좁히기 위해 올바로 동작하는 한 항상 가장 낮은 접근 수준을 부여해야한다
#### 톱레벨 클래스와 인터페이스에 부여할 수 있는 접근 수준 
- package-private : 해당 패키지 안에서만 사용 가능
	-  패키지 외부에서 사용할 이유가 없다면 package-private로 선언
	- public 일 필요가 없는 클래스의 접근 수준을 package-private 톱레벨 클래스로 좁히는것이 중요 - 내부구현에 속하도록 변경 되는 것이기 때문
- public : 공개 api
	- 공개 api 는 하위호환을 위해 영원히 관리해야함

한 클래스에서만 사용하는 package-private 톱 레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static 으로 중첩 시키면 바깥 클래스 하나에서만 접근할 수 있다
#### 멤버에 접근 수준을 부여할 때
클래스의 공개 api 를 세심히 설계한 후 그외 모든 멤버는 private로 만들기
그 다음 같은 패키지의 다른 클래스가 접근해햐 하는 멤버에 한하여 package-private으로 설정
package-private 에서 protected로 변경하면 접근 범위가 넓어짐
proteccted 멤버는 공개api임으로 영원이 지원되어야하고 내부 동작방식을 api 문서에 적어서 공개해야할 수도 있음
따라서 protected 멤버의 수는 적을 수록 좋음
### 멤버 접근성을 좁히지 못하게 방해하는 제약
상위 클래스의 메서드를 재정의 할때는 접근 수준을 상위 클래스에서보다 좁게 설정 불가능
리스코프 치환 원칙 때문
상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙
이를 어기면 컴파일시 에러 발생
### public class의 인스턴스 필드는 public을 지양할 것
- 필드와 관련한 것에 대한 불변식을 보장 할 수 없게 됨 
- 일반적으로 스레드 세이프 하지 못함
- 상수는 public static final 필드로 공개해도 좋음(이름은 알파벳 대문자, 띄어쓰기는 _ 로 처리)
- 상수는 기본 타입값이나 불변객체를 참조해야한다
- public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안됨

------
# Item16 public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

public class에서는 패키지 내부의 데이터를 private로 선언하고 public getter를 추가하는게 좋다.
#### 접근자 메서드를 생성할때의 장점
패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 바꿀 수 있는 유연성을 얻을 수 있다
public class의 필드가 불변이라면 직접 노출할 때의 단점은 줄지만 api를 변경하지 않고는 표현방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점을 보완해줌
단 불변식은 보장할 수 있다
#### 데이터 필드를 노출해도 괜찮은 경우
- package-private 클래스 혹은 private 중첩 클래스
	- 접근자 방식보다 훨씬 깔끔
클라이언트 코드가 클래스 내부표현에 묶이지만 패키지 또는 외부클래스 안에서 동작하기에 수정범위가 적기 때문

------
# Item17 변경 가능성을 최소화하라
### 불변 클래스란
인스턴스의 내부 값을 수정할 수 없는 클래스
객체가 파괴되는 순간까지 달라지지 않음
가변 클래스보다 구현하고 사용하가 쉬우며 오류가 생길 여지도 적고 안전함
### 불변클래스의 5가지 규칙
1. 객체의 상태를 변경하는 메서드를 제공하지 않는다
2. 클래스를 확장할 수 없도록 한다
3. 모든 필드를 final로  선언한다
4. 모든 필드를 private로 선언한다
5. 자신 외에는 내부의 가변 컴포넌트에 접근 할 수 없도록 한다.
+ 메서드 명명 규칙 : 동사대신 전치사를 사용할 것

불변객체는 모든 생성자가 클래스 불변식을 보장한다면 사용하는 프로그래머가 다른 노력을 하지 않더라도 불변성을 보장한다
#### 불변 객체의 장점
- 스레드 안전하여 따로 동기화 할 필요가없다
	- 여러 스레드가 사용해도 훼손되지 않으므로 스레드 세이프 하게 만드는 가장 쉬운 방법
- 안심하고 공유할 수 있다
	- 한번 만든 인스턴스를 최대한 재활용하기를 권한다
		- 방법
		- 상수로 제공
		- 정적팩터리 제공
- 방어적 복사가 필요없다
- 불변 객체끼리는 내부 데이터를 공유 할 수 있다
- 객체를 만들 때 다른 불변 객체를 구성요소로 사용하면 이점이 많다
- 그 자체로 실패 원자성을 제공한다
#### 불변객체의 단점
- 값이 다르면 반드시 독립된 객체로 만들어야 한다
	- 이때 값이 가지 수가 많으면 큰 비용을 처리해야 한다
	- 객체를 만드는 과정이 복잡하면 성능문제가 불거진다
		- 다단계 연산을 예측하여 기본 기능으로 제공
		- 가변 동반클래스 제공(string 은 StringBuilder)
#### 규칙을 완화해서...
불변 객체지만 큰 계산 비용이 필요한 필드는 호출 될때 계산하여 final이 아닌 필드에 캐시하여 활용하기도 함 - 불변이기에 항상 같은 값을 보장함
### 불변으로 만들 수 없는 클래스인 경우
변경할 수 있는 부분을 최소화 - 개체 예측이 쉬워지고 오류가 발생할 가능성이 줄어든다
합당한 이유가 없다면 모든 필드는 private final
생성자는 불변식 설정이 모두 완료된 초기화가 완벽히 끝난 객체를 생성해야한다
객체를 재활용할 목적으로 초기화하는 메서드도 안된다 복잡성만 커지고 이점은 없다

-------
# Item18 상속보다는 컴포지션을 사용하라

>상속은 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 생성하게 된다.
>특히 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.
### 메서드 호출과는 달리 상속은 캡슐화를 깨뜨린다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
- 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있고, 그 여파로 하위 클래스가 오동작할 수 있다
#### 하위 클래스가 깨지기 쉬운 이유
메서드 재정의 시
- 상위 클래스의 메서드 동작을 다시 구현하는 것은 어렵고, 시간도 더 들고, 오류를 내거나 성능을 떨어뜨릴 수도 있다. 또한 하위 클래스에서는 접근할 수 없는 private 필드를 써야 하는 상황이라면 이 방식으로는 구현자체가 불가능하다.
- 릴리스에서 상위 클래스에 새로운 메서드를 추가한 경우
	- 재정의 하지 못한 새롭게 추가된 메서드를 통해서 허용되지 않은 동작을 수행할 수 있게 될 수도 있다.
새로운 메서드 추가시
- 기존 하위 클래스에 추가한 메서드와 새로 상위 클래스에 추가된 메서드의 시그니처가 같고 반환 타입이 다르다면 컴파일조차 되지 않는다.
- 메서드 재정의 시와 같은 문제 발생
- 더불어 하위 클래스의 메서드는 상위 클래스에 새롭게 추가된 메서드가 요구하는 규약을 지키지 못할 가능성이 크다.
### 컴포지션(composition, 구성)
- 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻
- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하면 된다.
- 새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.
    - 이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.
    - 그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.
#### 상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야한다
- 클래스 B 가 클래스 A의 is-a관계 일때만 클래스 A 를 상속해야함
컴포지션을 써야할 상황에서 상속을 사용하는 건 내부구현을 불필요하게 노출하는 것
- 결과 api가 내부 구현에 묶이고 그 클래스이 성능도 영원히 제한됨
- 클라이언트가 노출된 내부에 직접 접근할 수 있어 사용자를 혼란 스럽게함
- 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 불변식을 해칠 수 있음

-------
# Item19 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 상속을 고려한 설계와 문서화
- 메서드를 재정의하려면 어떤 일이 일어나는지를 정확히 정리하여 문서로 남겨야 한다. 
좋은 API 문서란 '어떻게'가 아닌 '무엇'을 하는지를 설명해야 하는 것과 대조적인 방식
원인은 상속이 캡슐화를 해치기 때문
### 상속용 클래스 설계
- 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 '유일'하다.
- protected 메서드 수는 가능한 한 적어야 하되 너무 적게 노출해서 상속으로 얻는 이점을 없애지 않도록 주의해야 한다.
- 하위 클래스를 만들면서 필요 여부에 따라 처음 설계의 protected 멤버와 private 멤버가 변할 수 있다
- 문서화한 내부 사용 패턴과 protected 메서드와 필드를 구현하면서 선택한 결정을 영원히 책임져야 한다.
- 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출되기에  생성자는 재정의 메서드를 호출해서는 안 된다.
### clone과 readObject 메서드
- clone과 readObject 메서드는 생성자와 비슷하게 새로운 객체를 만들기 때문에 제약이 비슷하다.
- clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

final도 아니고 상속용으로 설계되지 않은 구체 클래스의 상속은 막아야 한다
#### 상속을 금지하는 두 가지 방법
- 클래스를 final로 선언하는 것이다.
- 모든 생성자를 private이나 package-private으로 선언하고 public 정적 팩터리 생성
#### 상속을 허용해야겠다면...
- 클래스 내부에서는 재정의 가능 메서드를 사용하지 않게 만들고 문서로 남겨야 한다.
    - 즉, 재정의 가능 메서드를 호출하는 자기사용 코드를 완벽히 제거해야 한다.
    - 이렇게 하면 상속해도 그리 위험하지 않다. 메서드를 재정의해도 다른 메서드의 동작에 아무런 영향을 주지 않기 때문이다.

--------
# Item20 추상 클래스보다는 인터페이스를 우선하라
자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상 클래스, 이렇게 두 가지다.
자바 8부터 인터페이스도 디폴트 메서드(default method)를 제공할 수 있게 되었다.
따라서, 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

인터페이스를 우선 해야하는 이유
- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현 할 수 있는 반면 새로운 추상 클래스를 끼워넣기는 어려움
- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이지만 추상 클래스는 불가능함
	- 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.
- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
	- 클래스로 구현 하고자 한다면 속성이 n개라면 지원해야 할 조합의 수는 2^n 개가 되는 조합 폭발(combinatorial explosion)현상이 발생할 수 있음
	- 거대한 클래스 계층구조에는 공통 기능을 정의해놓은 타입이 없으니, 자칫 매개변수 타입만 다른 메서드들을 수없이 많이 가진 거대한 클래스를 낳을 수 있다.
- 래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.
	- 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속뿐이다.
	- 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.

> 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공해줄 수 있다.
> 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그를 붙여 문서화해야 한다.
### 템플릿 메서드 패턴
- 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법.
    - 인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다
#### 단순 구현(simple implementation)
- 단순 구현도 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니란 점이 다르다.
- 쉽게 말하면, 동작하는 가장 단순한 구현이다. 이러한 단순 구현은 그대로 써도 되고 필요에 맞게 확장해도 된다.