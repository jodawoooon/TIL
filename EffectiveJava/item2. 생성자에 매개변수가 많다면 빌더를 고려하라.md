### item2. 생성자에 매개변수가 많다면 빌더를 고려하라
> Effective Java 공부

정적 팩토리와 생성자에는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 단점이 있다. 

<br>

### 점층적 생성자 패턴

주로 가장 많이 사용했던 것은 **점층적 생성자 패턴**이다. 필수 매개변수만 받는 생성자부터 추가적으로 선택 매개변수를 1개받는 생성자, 2개 받는 생성자 ... 형태로 선택 매개변수를 전부 받는 생성자까지 늘려가는 방식이다.


```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}

```

이는 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 각 값의 의미가 헷갈리고 매개변수가 몇 개인지도 주의해야한다. 실수로 매개변수의 순서를 바꿔도 컴파일러가 거르지 못하기 때문에 올바르지 않은 결과가 나오게 된다.

<br>

### 자바 빈즈 패턴

다음은 **자바 빈즈 패턴**(`JavaBeans Pattern`)이다. 이는 매개변수가 없는 생성자로 객체를 만든 후 Setter 메소드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.


```java
public class NutritionFacts {

    private int servingSize = -1; //필수 : 기본값 없음
    private int servings = -1; //필수 : 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {};
    
    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}

```


코드가 길어지긴 하지만 인스턴스를 만들기 쉽고 읽기 쉬워졌다. 다만 심각한 단점이 하나 있다. 
자바빈즈 패턴에서는 객체를 만들기 위해 여러 메소드를 호출해야 하고 객체가 완전히 생성되기 전까지는 **일관성**이 무너진 상태가 된다.

앞선 점층적 생성자 패턴에서는 매개변수들이 유효한지 생성자에서만 확인하면 일관성 유지가 가능했지만 자바빈즈 패턴에 경우에는 이것이 불가능하다. 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없고 쓰레드 안전성을 위해 추가적 작업(`수동으로 freeze`)이 필요하다.

<br>

### 빌더 패턴

이러한 점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 겸비한 것이 바로 **빌더 패턴**(`Builder Pattern`)이다. 

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    static static class Builder {
        
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
        
        // 선택 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
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
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
    
    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    };
}
```

필요한 객체를 직접 만드는 대신 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다. 그리고 빌더 객체가 제공하는 세터 메소드로 선택 매개변수들을 설정하고 `build` 메소드를 호출해 객체를 얻는다.

위의 코드와 `NutritionFacts` 클래스는 불변이며 모든 매개변수의 기본값을 모아뒀다. 빌더의 `setter` 메소드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(240).sodium(35).carbohydrate(27).build();
```

이와 같이 빌더패턴은 쓰기 쉽고 읽기 쉽다.

빌더 패턴에 장점만 있지는 않다. 객체를 만들려면 그에 앞서 빌더부터 만들어야 하므로 성능에 민감한 상황에서는 문제가 될 수 있다. 또한 점층적 생성자 패턴보다는 코드가 장황해서 매개 변수가 4개 이상은 되어야 값어치를 한다.

<br>

**생성자나 정적 팩토리가 처리해야 할 매개변수가 많다면 빌더패턴을 선택하는게 낫다.** 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기 간결하고 자바빈즈보다 안전하다.