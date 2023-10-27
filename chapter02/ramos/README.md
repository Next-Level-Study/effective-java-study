## 2장. 객체 생성과 파괴
DTO class, XxxUtils와 같은 클래스에서 정적 팩토리 메서드를 많이 사용하는데 Builder 패턴과 연계해서 사용한다면 더 효과적이라 생각된다.

DTO class의 경우, 필드가 많아진다면 특히 Builder가 유용하다.

일반적으로 네이밍 규칙은 `of`, `from`, `getInstance` 등과 같이 사용하는데 아래 예제가 이에 해당한다.

```java
public static Item1 of(String name, String email) {
    return Item1.builder()
        .name(name)
        .email(email)
        .build();
}
```

개인적으론 JPA Entity ↔ DTO class 간의 변환과 같은 케이스엔 `fromXXX`, `toXXX`와 같은 네이밍이 좋다고 생각한다.

다음은 간단한 유틸성 클래스에 대한 예제이다.

```java
package me.ramos.lambda.config;

import com.amazonaws.regions.Regions;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;

public class AWSConfig {

    // Item 4. 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때
    private AWSConfig() {
    }

    private static final String DYNAMODB_TABLE_NAME = "players";
    private static final Regions REGION = Regions.AP_NORTHEAST_2;

    private static AmazonDynamoDB amazonDynamoDB;

    // 멀티스레딩에 대한 방어적인 singleton이지만, eager initialization 방식이 더 낫겠다.
    public static synchronized AmazonDynamoDB initDynamoDbClient() {
        if (amazonDynamoDB == null) {
            amazonDynamoDB = AmazonDynamoDBClientBuilder.standard()
                    .withRegion(REGION)
                    .build();
        }
        return amazonDynamoDB;
    }

    /**
     * 이 예시에선 하나의 AWS Lambda를 위해 Connection을 관리할 목적으로 사용했기에
     * 이와 같이 구성하였다.
     * 단, table name이 바뀌지 않는다는 전제에만 유효하다.
     **/
    public static String getDynamodbTableName() {
        return DYNAMODB_TABLE_NAME;
    }
}
```

덧붙여, singleton 구현시 Java Reflection, Serialization/Deserialization으로 singleton 객체를 파괴할 수 있고 Reflection의 경우는 대응할 방법이 없다는 점에 유의하자.

직렬화/역직렬화의 문제는 반드시 `getInstance()`와 같은 메서드를 호출하도록 singleton을 구현하고 해당 객체가 enum으로 관리해도 무방하다면 enum으로 singleton을 보장하자.

사실 위 예제 코드는 Item 5의 내용이 지향하는 바를 충족하진 않는다. 단순히 하나의 AWS Lambda 내에서만 사용하고 있으니 이에 대응되는 DynamoDB의 table이 하나라서 변경이 덜하지만, 엄밀하게 따진다면 이 또한 유연하지가 않다. (OCP 원칙을 떠올려보자.) 좀 더 개선해본다면, Spring 환경이 아니니 환경변수를 통해 table name을 설정할 수 있도록 바꾸면 좋겠다.

---

```java
String s = new String("hello"); // 하지 말아야 할 극단적인 예시

String s = "hello"; // 개선된 버전
```

새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 이 방식을 사용한다면, JVM 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.

정적 팩토리 메서드를 제공하는 불변 클래스의 경우, 불필요한 객체 생성을 피할 수 있으니 적극적으로 활용하자.

또한, **박싱된 기본타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

---

item 7에서 느낀점이라면, 캐시나 callback과 같은 상황이 memory leak의 주범이기도 하지만 더 나아가 우리가 DB에서 데이터를 효율적으로 query하는 것도 필요하단 생각이 든다.

대량의 데이터를 읽어오는 query를 수행하는 상황을 생각해보자. DBMS(ex. MySQL)가 자신의 memory에 적재해서 결과를 던져주는데(당연히 DBMS 자체의 memory 사용량도 잘 생각해봐야 함), 이를 기반으로 WAS는 JPA Entity 라던지, MyBatis Mapper 라던지 객체에 데이터를 매핑해서 처리하기 때문에 이 과정에서 WAS 상에 생성되는 객체에 대해 명시적으로 참조를 해제하는 작업을 애플리케이션 내에서 수행했었나? 아마 그런 기억이 없을 것이다.

다른 방법이 생각이 나지 않다면, DB의 index를 활용한다던지 등의 효율적인 query를 수행하는게 최선이라고 생각한다.

---

`try-with-resouces`는 Java 7에서 등장했다. 이에 더해 Java 9에서 이 기능을 좀 더 개선했으니 알아두면 좋을 것 같다.

```java
// Java 9 이전
void tryWithResourcesByJava7() throws IOException {
    BufferedReader reader1 = new BufferedReader(new FileReader("test.txt"));
    try (BufferedReader reader2 = reader1) {
        // do something
    }
}

// Java 9 ~
void tryWithResourcesByJava9() throws IOException {
    BufferedReader reader = new BufferedReader(new FileReader("test.txt"));
try (reader) {
        // do something
    }
}
```

`try` 블럭에서 선언된 객체들에 대해서 `try` 문이 종료될 때 자동으로 자원을 해제해주는 기능이다. Java 9 이후에는 `try` 블럭 밖에서 선언한 변수를 가져와 사용할 수 있다.