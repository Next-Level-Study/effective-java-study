클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 적합하지 않다.
사용하는 자원에 따라 동작이 달라지는 클래스는 의존 객체 주입의 한 형태인 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하는 것이 좋다.

```java
// 의존 객체 주입은 유연성과 테스트 용이성을 높여준다.
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```
 이 방식은 자원의 개수, 의존 관계가 어떤지와 관계없이 잘 작동하며, 불변을 보장한다.
 또한 불변을 보장하여 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다. 
 의존 객체 주입은 생성자, 정적 팩토리, 빌더 모두에 똑같이 응용할 수 있다.
 또한 생정자에 팩터리 메서드 패턴을 구현한 자원 팩터리를 넘겨줄 수도 있다. `Supplier<T>` 가 좋은 예시이다. 
 의존 객체 주인의 단점
 - 유연성과 테스트 용이성을 개선해주긴 하지만 프로젝트에서는 코드를 어지럽게 만들기도 하는데 대거(Dagger), 주스(Guice), Spring 같은 프래임워크를 사용하면 해소 가능하다.
