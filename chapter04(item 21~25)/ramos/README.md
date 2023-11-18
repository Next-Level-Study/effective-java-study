## 4장. 클래스와 인터페이스
📌 [4장의 item 별 요약은 다음을 참고해주세요.](https://github.com/alanhakhyeonsong/LetsReadBooks/tree/master/Effective%20Java%203E/contents/chapter04)

// 흥미로운 이야기만 이 파일에 작성합니다.

### Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라
Item 21 내용 중 대표적인 사례를 직접 찾아보며 어떻게 구현되어 있는지 확인해보았다.

`Collections` 내부에 있는 `SynchronizedCollection`은 다음과 같다.

![스크린샷 2023-11-19 오전 2 07 14](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/fdd5da72-0b96-4d56-bd7d-7d21bb36ea71)

`removeIf`는 기본적으로 코드 구현이 되어 있는데 `SynchronizedCollection` 클래스는 동기화 처리를 해줘야하기 때문에 해당 부분을 `synchronized` 블럭으로 감싸서 대처하는 모습을 볼 수 있다.

![스크린샷 2023-11-19 오전 2 07 34](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/a9ad783e-ba93-42ce-abc1-8f2e85e270fe)

`SynchronizedCollection`가 처음 생성될 때 존재하지 않았던 인터페이스인데 이러한 부분을 `Collections` 에서 `default` 메서드로 정의해 두었다. 아래는 `Collections`에 `default` 메서드로 추가된 여러 개의 메소드 중 하나인 `removeIf` 메서드다.

![스크린샷 2023-11-19 오전 2 07 49](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/8ccc21b6-49f3-47e3-9911-af67c7c9268a)

`SynchronizedCollection` 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 `removeIf`를 호출하면 `ConcurrentModificationException`이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

### Item 24. 멤버 클래스는 되도록 static으로 만들라

147~148p를 보면 다음과 같은 내용이 나온다.

> **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 `static`을 붙여서 정적 멤버 클래스로 만들자.** `static`을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다. 앞서도 얘기했듯 이 참조를 저장하려면 시간과 공간이 소비된다. 더 심각한 문제는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다는 점이다(아이템 7). 참조가 눈에 보이지 않으니 문제의 원인을 찾기 어려워 때때로 심각한 상황을 초래하기도 한다.

```java
package me.ramos.eejava;

public class OuterReference {

    public class InnerClass {

    }

    public static class StaticInnerClass {

    }
}
```

// Java 18에서의 결과

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/f24756c1-9808-4b8e-91e2-5df6a9d31578)

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/3c35b7f8-e390-4a8e-97b6-809af3eefa2b)

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/ddc6d5d1-5e30-45e2-bdfa-693b8309967b)

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/0f801afd-dddc-492b-ae3a-b777f7143caa)

// Java 14에서의 결과

![1](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/d362cf15-75a0-48f1-89b1-536d9c87fbf1)

![2](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/aeff6d11-7be1-464d-9aa9-1e04058ba632)

![3](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/97063400-2b89-458f-b138-37ceb891d0b2)

`InnerClass`가 바깥 클래스인 `OuterReference`를 참조하고 있고 `StaticInnerClass`는 그렇지 않음을 확인할 수 있다. 이 사실은 곧 GC가 발생할 때 각각 다르게 동작하게 되는 결과를 낳는다. 정적 멤버 클래스는 외부 인스턴스 없이도 만들어질 수 있기 때문에 '외부 참조'가 존재하지 않게 되고, 이로 인해 일회용으로 사용된 바깥 클래스 객체는 더이상 내부 클래스 객체와 아무런 관계가 아니게 되어 정상적으로 GC 수거 대상이 되어 메모리 관리가 잘 된다.

좀 더 자세한 예제를 확인해보자.

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/3420d016-afb7-45d1-ab8a-0d23bcc90b49)

이런게 가능할까?