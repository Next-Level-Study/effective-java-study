# Item21 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 이전의 인터페이스를 보완하기 위해 기존 인터페이스에 메서드를 추가할 수 있는 디폴트 메서드가 등장

## 디폴트 메서드를 만들어서 문제가 생긴 사례
> 디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재 정의 하지 않은 모든 클래스에서 디폴트 구현이 쓰인다. 여기서 디폴트 메서드는 구현 클래스에 대해 아무것도 모르고 합의없이 삽입될 뿐인 메서드이기에 문제가 발생한다
- 모든 상황에서 불변식을 해치지 않은 디폴트 메서드를 작성하기 어려워서 문제가 발생 됨
- 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬수 있다

Apache commons 라이브러리의 SynchronizedCollection 클래스는 모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬랙션 객체에 기능을 위임하는 래퍼클래스인데 removeIf 는 동기화에 대해 몰라 락 객체를 사용할 수 없었다([4.4 버전에서야 패치됨](https://commons.apache.org/proper/commons-collections/apidocs/src-html/org/apache/commons/collections4/collection/SynchronizedCollection.html#line.193) [git](https://github.com/apache/commons-collections/commit/bd76c282598967655d12697b6b1aba90b1da735b#diff-87abc868f14075451937bcf4bd1d6f2f67f9d2235743db8f2c68f3b59ffcbc8a))
이런 문제를 예방하기 위해 구현한 인터페이스의 디폴트 메서드를 재정의하고 디폴트 메서드를 호출하기 전에 필요한 작업을 수행해야함

### 디폴트 메서드를 만들때 주의해야 할 점
-  기존 구현체에 런타임 오류를 일으킬 수 있기 때문에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해아한다
- 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아니므로 인터페이스를 설계할 때는 세심한 주의가 필요하다
- 새로운 인터페이스라면 릴리즈 전에 테스트를 해야한다

# Item22 인터페이스는 타입을 정의하는 용도로만 사용하라

> 인터페이스는 클래스의 인스턴스를 참조할 수 있는 타입 역할을 함
> 클래스가 인스턴스를 구현한다는 것은 인스턴스로 무엇을 할 수 있는 지를 클라이언트에 얘기 해주는 것
## 이 지침의 맞지 않는 예
### 상수 인터페이스
- 메서드 없이 상수를 뜻하는 static final 필드로만 가득찬 인터페이스
- 인터페이스를 잘못 사용한 예

#### 상수 인터페이스의 문제점
- 상수는 외부 인터페이스가 아닌 내부 구현임으로 내부 구현을 클래스의 api로 노출하는 행위가 됨
- 사용자에게 혼란 제공
- 클라이언트 코드가 내부구현인 상수에 종속
- 다음 릴리즈에서 쓰지 않게 되도 바이너리 호환성을 위해 제거 불가능

#### 상수 인터페이스의 대안
- 클래스나 인터페이스 자체에 추가
- 열거 타입으로 만들어 공개
- 인스턴스화할 수 없느 유틸리티 클래스에 담아 공개
	- 유틸리티 클래스에서 사용한다면 클래스 이름까지명시
	- 빈번하다면 정적임포트

# Item23 태그 달린 클래스보다는 클래스 계층구조를 활용하라

## 태그 달린 클래스의 단점
- 열거 타입 선언, 태그 필드, switch 문 등 쓸데없는 코드가 많아
- 여러 구현이 혼합되어 가독성이 나쁘다
- 다른 의미를 위한 코드가 함께 해 메모리를 많이 사용한다.
- 쓰이지 않는  필드까지 생성자에서 초기화 해야한다
- 다른 의미를 추가하려면 코드를 수정해야한다
- 인스턴스 타입만으로는 현재 나타내는 의미를 알 길이 없음
>태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적임

## 대체방안
### 클래스 계층 구조(Subtyping)
태그 달린 클래스를 계층 구조로 바꾸는 방법
1. 계층 구조의 루트가 될 추상 클래스 정의
2. 동작이 달라지는 메서드를 로트 클래스의 추상 메서드로 선언
3. 태그와 상관없이 동작이 일정한 메서드를 루트클래스에 일반 메서드로 추가
4. 공통 데이터 필드를 루트클래스에 추가
5. 루트 클래스를 확장한 구체클래스를 의미별로 정의
6. 각 하위 클래스에 알맞은 데이터 필드를 추가
결과로 태그 달린 클래스의 단점은 사라지고 간결하고 명확하며 쓸데없는 코드들이 삭제됐다.
나중에 다른 프로그래머가 독집적으로 계층구조를 확장할 수 있다
또한 타입사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다

# Item24 멤버 클래스는 되도록 static으로 만들라

# 중첩클래스
>다른 클래스 안에 정의된 클래스를 말하고 자신의 바깥 클래스에서만 쓰여야한다

## 종류
### 정적멤버클래스
- 다른 클래스 안에 선언되고 바깥 클래스에 private 멤버에 접근 가능
- 다른 정적 멤버와 간은 접근 규칙을 적용받음
- 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 사용됨
```java
public class StaticMemberClassEx {  
	private String test = "test";  
	// enum은 암시적 static
	public enum Number {  
		ONE, TWO, THREE  
	}  
	private static class PrivateSample {  
		public static void method() {  
			StaticMemberClassEx outerClass = new StaticMemberClassEx();  
			System.out.println("private" + outerClass.test); // 바깥 클래스인 StaticMemberClassEx의 private 멤버 접근  
		}  
	}  
	public static class PublicSample {  
		public static void method() {  
			StaticMemberClassEx outerClass = new StaticMemberClassEx();  
			System.out.println("public" + outerClass.test); // 바깥 클래스인 StaticMemberClassEx의 private 멤버 접근  
			StaticMemberClassEx.PrivateSample.method();  
		}  
	}  
}  
  
class TestMain {  
	public static void main(String[] args) {  
		System.out.println(StaticMemberClassEx.Number.ONE);  
		StaticMemberClassEx.PublicSample.method(); // public 접근 가능 private 접근불가 
	}  
}
```
### 비정적 멤버 클래스
- 바깥 인스턴스 없이는 생성이 불가능 해서 중첩 클래스의 인스턴스가 바깥 인스턴스와 독집적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야함
- 바깥 클래스의 인스턴스와 암묵적으로 연결됨
	- 인스턴스 메서드에서 정규화된 this(클래스명.this)를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있음
	- 클래스가 인스턴스화 될때 바깥 인스턴스와의 관계가 확립되고 변경 불가능함
	- 관계정보는 비정적 멤버클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하고 생성시간도 더 걸림
- 어댑터를 정의할 때 자주 쓰임
- 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없으면 static을 붙여 정적 멤버 클래스로 만들것
	- static 생략시 숨은 외부참조 생성
		- 시간과 공간 소비
		- 가비지 컬랙션이 바깥 클래스 인스턴스를 수거하지 못하는 메모리 누수 발생가능성이 있음
- public이나 protected 멤버라면 공개 API가 되어 향후 릴리즈에서 static을 붙이면 하위 호환성이 깨짐
```java
public class MemberClassEx {  
	public class MemberClass {  
		public void callTestClassMethod() {  
			// 바깥 클래스의 메소드 호출  
			MemberClassEx.this.testMethod();  
		}  
	}  
	public void testMethod() {  
		System.out.println("hello world");  
	}  
}  
  
class MemberClassExTest {  
	public static void main(String[] args) {  
		MemberClassEx a = new MemberClassEx();  
		MemberClassEx.MemberClass b = a.new MemberClass();  
	}  
}
```
### 익명 클래스
- 바깥 클래스의 멤버가 아님
- 선언과 동시에 인스턴스 생성
- 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조 가능
- 상수 이외의 정적필드를 가질 수 없음
- 제약
	- 선언 시점에서만 인스턴스 생성 가능
	- instanceof 검사 또는 클래스 이름이 필요한 작업은 수행 불가능
	- 여러 인터페이스 구현 불가
	- 인터페이스를 구현하는 동시에 다른 클래스 상속 불가
	- 사용하는 클라이언트는 익명클래스가 상위타입에서 상속한 멤버 외에는 호출 할 수 없음
	- 표현식 중간에 등장하여 짧지 않으면 가독성 저하
- 주 사용처
	- 작은 함수 객체나 처리객체를 만드는데 사용 -> 람다
	- 정적 팩터리 메서드를 구현시
```java
public class AnonymousClassEx {  
	public void test() {  
		System.out.println("test");  
	}    
}  
  
class AnonymousClassExTest{  
	public static void main(String[] args) {  
	// 익명 클래스 : 클래스 정의와 객체화를 동시에. 일회성으로 사용  
		AnonymousClassEx a = new AnonymousClassEx() {  
			@Override  
				public void test() {  
					System.out.println("test-test");  
				}  
		};
		a.test();  
	}  
}
```
### 지역 클래스
- 가장 드물게 사용됨
- 지역 변수를 선언할 수 있는 곳이면 어디든 선언 가능, 유효범위 == 지역변수의 유효범위
- 멤버클래스처럼 이름이 있고 반복해서 사용가능
- 비정적 문맥에서 사용될 때만 바깥 인스턴스 참조 가능
- 정적멤버는 가질 수 없으며 가독성을 위해 짧게 작성
```java
public class LocalClassEx {  
	private int x = 0;  
	public void test() {  
	// 지역변수처럼 선언  
		class LocalClass {  
			// private static int staticNumber; // 정적 멤버 가질 수 없음  
	  
			public void print() {  
				// 비정적 문맥에서 사용될 때만 바깥 인스턴스 참조 가능 
				System.out.println(x);  
			}  
		}  
		LocalClass a = new LocalClass(); // 이름이 있고 반복해서 사용 가능
		LocalClass b = new LocalClass();   
	}  
}
```

# Item25 톱레벨 클래스는 한 파일에 하나만 담으라

### 톱레벨 클래스가 한 파일에 여러개가 있다면...
자바 컴파일러는 불평을 하지는 않지만 득이 없을 뿐더러 심각한 위험을 감수해야함
한 클래스를 여러가지로 정의할 수 있으며 그중 어느 것을 사용할지는 어떤 소스 파일을 먼저 컴파일 하냐에 따라 달라지기 때문

### 톱레벨 클래스를 한 파일에 담고 싶다면... 
정적 멤버 클래스를 사용하는 방법을 고려
다른 클래스에 딸린 부차 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 더 나음
읽기 좋고 접근범위도 최소로 관리할 수 있기 때문
