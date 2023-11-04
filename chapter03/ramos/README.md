## 3장. 모든 객체의 공통 메서드
📌 [3장의 item 별 요약은 다음을 참고해주세요.](https://github.com/alanhakhyeonsong/LetsReadBooks/tree/master/Effective%20Java%203E/contents/chapter03)

### Item 14. Comparable을 구현할지 고려하라.
91p 내용 중 다음과 같은 문장에 대해 의문이 들어 예제를 짜봤다.

> Java 7부터는 상황이 변했다. 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 `compare`를 이용하면 되는 것이다. `compareTo` 메서드에서 관계 연산자 `<`와 `>`를 사용하는 이전 방식은 거추장스럽고 오류를 유발하니, 이제는 추천하지 않는다.

```java
package eejava;

public class MyInteger {

    public static class CompareMyInteger implements Comparable<CompareMyInteger> {

        private Integer integer;

        @Override
        public int compareTo(CompareMyInteger o) {
            return Integer.compare(this.integer, o.integer);
        }
    }

    public static class RelationalMyInteger implements Comparable<RelationalMyInteger> {

        private Integer integer;

        @Override
        public int compareTo(RelationalMyInteger o) {
            if (this.integer == o.integer) {
                return 0;
            } else if (this.integer > o.integer) {
                return 1;
            }
            return -1;
        }
    }
}
```

분명 `Integer integer` 처럼 Wrapper 클래스로 만들어 뒀으니 테스트 코드를 통해 어떤 일이 일어나는지 확인해보자.

```java
package eejava;

import eejava.MyInteger.CompareMyInteger;
import eejava.MyInteger.RelationalMyInteger;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

class MyIntegerTest {

    @DisplayName("compare 비교 시 NPE 발생")
    @Test
    void compareTo() throws Exception {
        //given
        CompareMyInteger myInteger1 = new CompareMyInteger();
        CompareMyInteger myInteger2 = new CompareMyInteger();

        //when, then
        Assertions.assertThrows(NullPointerException.class, () -> myInteger1.compareTo(myInteger2));
    }

    @DisplayName("관계 연산자 비교 시 두 객체의 비교 필드가 null인 경우 의도하지 않은 결과가 발생할 수 있다.")
    @Test
    void relationalCompare() throws Exception {
        //given
        RelationalMyInteger myInteger1 = new RelationalMyInteger();
        RelationalMyInteger myInteger2 = new RelationalMyInteger();

        //when, then
        Assertions.assertEquals(0, myInteger1.compareTo(myInteger2));
    }
}
```

![image](https://github.com/Next-Level-Study/effective-java-study/assets/60968342/dc33a57d-191c-4d7a-8e22-9651a739b3ea)

자세히 설명되어 있진 않았지만, `null` 과 관련된 오류가 있으니 책에서 해당 문장으로 언급하지 않았을까 싶다.