### 점층적 생성자 패턴
필수 매개변수를 받는 생성자 1개, 그리고 선택매개변수를 하나씩 늘려가며 생성자를 만드는 패턴


```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) { // 필수 매개변수
        this(servingSize, servings, 0); //다음 생성자를 호출함
    }

    public NutritionFacts(int servingSize, int servings, int calories) { // 선택 매개변수 calories 추가
        this(servingSize, servings, calories, 0);
    }

    ...

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}
```

이중에 원하는 것을 골라 쓸 수 있도록 만들어 놓은 것인데 단점이 있다
1. 원치 않은 매개 변수에도 값을 지정해 주어야 한다.
2. 복잡하고 읽기 어렵다
3. 클라이언트의 실수로 매개변수가 바뀌었을 때 오류를 찾기 어렵다.

### 대안 자바 빈즈 패턴
매개변수가 없는 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수 값을 설정하는 방식

```java
@Setter
public class NutritionFacts {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}
}
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

점층적 생성자의 단점을 보완
하지만 객체를 하나를 만들기 위해 메서드를 여러개 호출해야하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상테에 놓임

단점 보완을 위해 생성이 끝난 객체를 freeze 메서드를 이용해 수동으로 얼려줘야함

### 빌더패턴
점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```

사용 예 - 쓰기 쉽고 읽기 쉽다!
```java
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
      .calories(100)
      .sodium(35)
      .carbohydrate(30)
      .build();
```
NutritionFacts 클래스는 불변이며, 모든 매개변수의 기본값을 한 곳에 모아 뒀다.
이 빌더의 Setter 메서드는 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.(method chaining)
잘못된 매개변수를 체크하기 위해 불변식을 넣어 체크할 수도 있다.

빌더 패턴은 계층적으로 설계된 클래스와 사용하기도 좋다.
하위 클래스의 메서드가 상위 클래스의 매서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 공변 반환타이핑을 이용하여 클라이언트가 형변환에 신경을 쓰지 않을수 있게 해줌
빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.
성능 객체를 만들려면 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 크진 않지만, 성능에 민감한 상황에서는 문제가 될 수 있다.