## 7장. 람다와 스트림
📌 [7장의 item별 요약은 다음을 참고해주세요](https://github.com/alanhakhyeonsong/LetsReadBooks/tree/master/Effective%20Java%203E/contents/chapter07)

### Java에서의 고차함수?
**고차 함수는 함수를 인수로 전달받거나 함수를 반환하는 함수를 말한다.**

Java 8부터 함수가 1급 객체가 되었는데, 이말은 함수는 다른 함수의 인자가 되거나 반환형이 되거나 객체로 저장될 수 있다는 것을 의미한다.

이런 함수형 프로그래밍이 가능한 TypeScript 코드 예시를 보자.

```typescript
function i18nHello (helloText: string): (name: string) => string {
  return (name) => { return helloText + " " + name };
}

let en = i18nHello("Hello🇺🇸");
let ko = i18nHello("안녕🇰🇷");
let fr = i18nHello("Bonsoir🇫🇷");
let es = i18nHello("Hola🇪🇸");

console.log(en("Ramos"));
console.log(ko("Ramos"));
console.log(fr("Ramos"));
console.log(es("Ramos"));

/* 실행 결과
Hello🇺🇸 Ramos
안녕🇰🇷 Ramos
Bonsoir🇫🇷 Ramos
Hola🇪🇸 Ramos
こんにちは🇯🇵 Ramos
*/
```

위 코드를 Java 8로 바꾸면 아래와 같다.

```java
Function<String, Function<String, String>> i18nHello = (helloText) -> {
    return (name) -> {
        return helloText + " " + name;
    };
};

Function<String, String> en = i18nHello.apply("Hello🇺🇸");
Function<String, String> ko = i18nHello.apply("안녕🇰🇷");
Function<String, String> fr = i18nHello.apply("Bonsoir🇫🇷");
Function<String, String> es = i18nHello.apply("Hola🇪🇸");
Function<String, String> jp = i18nHello.apply("こんにちは🇯🇵");

System.out.println(en.apply("Ramos"));
System.out.println(ko.apply("Ramos"));
System.out.println(fr.apply("Ramos"));
System.out.println(es.apply("Ramos"));
System.out.println(jp.apply("Ramos"));

/* 실행 결과
Hello🇺🇸 Ramos
안녕🇰🇷 Ramos
Bonsoir🇫🇷 Ramos
Hola🇪🇸 Ramos
こんにちは🇯🇵 Ramos
*/
```

Java로 짜보니 코드가 뭔가 불편하다. 아무래도 `Function` 인터페이스를 명시해야 하므로 그런 것 같다.

- 일단 JavaScript/TypeScript와 마찬가지로 Java에서도 클로저의 개념 역시 사용한다.
  - `helloText` 처럼 외부에서 접근하고 스코프가 종료돼도 계속 접근할 수 있는 것을 클로저라고 해요:)

`Function` 인터페이스를 확인해보자.

![image](https://github.com/Next-Level-Study/effective-java-study/assets/60968342/4dbc5c75-75a9-48cb-8183-523b53d776ee)

`@FunctionalInterface`인 `Function` 인터페이스의 제네릭 타입을 보면,

- `T`: 함수에 들어갈 input type
- `R`: 함수가 반환하는 result type

`apply`는 말 그대로 이 `Function`을 실행하는 것이다. 또한 default method인 `compose`, `andThen`, `identity`는 `Function`을 리턴한다.

`R` type에 `Function`을 중첩해서 넣을 수 있다. 이 말은 곧 함수를 객체로 사용할 수 있다는 의미다. 이는 currying을 사용하는 함수형 프로그래밍 기법의 특징이 들어있다.

이런 특징을 활용하여 예시를 만들어 보았던 것이다.

앞선 예제에서 `i18nHello`와 `en`, `ko` 같은 함수를 분리해서 체이닝 시킨 것이 바로 커링이다. 다시 말해, **x, y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다.** g라는 함수 역시 하나의 인수를 받는 함수를 반환한다. 함수 g와 원래 함수 f가 최종적으로 반환하는 값은 같다.

즉, `f(x, y) = (g(x))(y)`가 성립한다.

(고교 수학에서의 함수의 개념을 생각해보면 이와 같다.)