# item2. 생성자에 매개변수가 많다면 빌더를 고려하라. 

이전에는  `1)점층적 생성자 패턴(telescoping constructor pattern)` 을 사용했다. 

점층적 생성자 패턴 - 확장하기 어렵다.
```java
public class NutiritionFacts {
    private final int servingSize;  // (ml, 1회 제공량)    필수
    private final int servings;     // (회, 총 n회 제공량)  필수     
    private final int calories;     // (1회 제공량당)       선택 
    private final int fat;          // (g/1회 제공량)      선택
    private final int sodium;       // (mg/1회 제공량)     선택 
    private final int carbohydrate; // (g/1회 제공량)      선택

    public NutritionFacts(int servingSize, int servings){
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories){
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat){
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium){
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate){
        this(servingSize, servings, calories, fat, sodium, carbohydrate, 0);
    }
}

```
위 클래스의 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다. 

```java
NutritionFacts cocacola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
**점청적 생성자 패턴도 쓸 수 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다**

클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고, 결국 런타임에 엉뚱한 동작을 하게 된다. 



`2)자바빈즈 패턴(JavaBeans pattern)` 도 있다. 
생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다. 

자바빈패턴 - 일관성이 깨지고, 불변으로 만들 수 없다.
```java
public class NutiritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize = -1;   //필수; 기본값 없음
    private int servings    = -1;   //필수; 기본값 없음
    private int calaries    = 0;
    private int fat         = 0;
    private int sodium      = 0;
    private int carbohydrate= 0;

    public NutiritionFacts() { }

    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calaries = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```

**자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.**

**일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들수 없으며**, 스레드의 안정성을 얻으려면 프로그래머가 추가작업을 해줘야 한다.


그래서 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 것이 `빌더패턴(Builder pattern)`이다. 

빌더 패턴- 점층적 생성자와 자바빈즈 패턴의 장점만 취했다.
```java
public class NutiritionFacts {
    private final int servingSize;  // (ml, 1회 제공량)    필수
    private final int servings;     // (회, 총 n회 제공량)  필수     
    private final int calories;     // (1회 제공량당)       선택 
    private final int fat;          // (g/1회 제공량)      선택
    private final int sodium;       // (mg/1회 제공량)     선택 
    private final int carbohydrate; // (g/1회 제공량)      선택

    public static class Builder {
        // 필수 매개변수 
        private final int servingSize;
        private final int servings; 

        // 선택 매개변수 - 기본값으로 초기화 한다.
        private final int calories      = 0;
        private final int fat           = 0;
        private final int sodium        = 0;
        private final int carbohydrate  = 0;

        public Builder(int servingSize, int servings){
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) { calaries = val; return this;}
        public Builder fat(int val) { fat = val; return this;}
        public Builder sodium(int val) { sodium = val; return this;}
        public Builder carbohydrate(int val) { carbohydrate = val; return this;}
        public NutritionFacts build() {
            return new NutiritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder){
        servingSize = builder.servingSize;
        servings    = builder.servings;
        calories    = builder.calories;
        fat         = builder.fat;
        sodium      = builder.sodium;
        carbohydrate= builder.carbohydrate;
    }
}
```

빌더 패턴은 이렇게 호출할 수 있다. 
```java
NutiritionFacts cocaCola = new NutiritionFacts.Builder(240, 8)
                                        .calories(100)
                                        .sodium(35)
                                        .carbohydrate(27)
                                        .build();
```

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.** 

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        abstract Pizza build();

        // 하위클래스는 이 메세더를 재정의 하여
        // "this"를 반환하도록 해야한다.
        protected abstract T self();
    }
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```
1) Pizza.Builder클래스는 재귀적 타입 한정을 이용하는 제너릭 타입이다. 
2) 여기에 추상메서드인 self를 더해 하위 클래스에서는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다. 

```java
public class NyPizza extends Pizza{

    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder>{

        private final Size size;

        public Builder(Size size){
            this.size = Objects.requireNonNull(size);
        }
        @Override public NyPizza build() {
            return new NyPizza(this);
        }
        @Override protected Builder self() { return this; };
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```java
public class Calzone extends Pizza{
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false;

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
        @Override public Calzone build() {
            return new Calzone(this);
        }
        @Override protected Builder self() { return  this; }
    }
    private Calzone(Builder builder){
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```
1. 각 하위 클래스의 빌더가 정의한 build 메서드는 해당 구체 하위 클래스를 반환하도록 선언한다. 

```
핵심정리 
생성나자 정적 팩터리가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더욱 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨신 간결하고, 자바 빈즈보다 훨씬 안전하다.
```