# item42. 익명 클래스보다는 람다를 사용하라

추상메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용 
-> 이런 인터페이스의 인스턴스를 함수객체(function object) 라고 하여 특정 함수나 동작을 나타내는데 사용했다. 

예를들어 문자열의 길이 순으로 정렬할 때 아래처럼 사용했다(Me too)
```java
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length())
    }
})
```

java8 에서 추상 메서드 하나짜리는 람다식으로 만들어서 사용할 수 있게 되었다. 

```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.lenght(), s2.length()))
```
- 컴파일러가 문맥을 살펴서 타입을 추론해준다 
- 상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 그럴 때는 프로그래머가 직접 명시해야 한다 
- **타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입을 생략하자**


위 코드를 더 간결하게 만들 수 있다. 
```java
Collections.sort(words, comparingInt(String::length))
```

List 인터페이스에 추가된 sort 메소드를 이용하면 더 짧아진다 
```java
words.sort(comparingInt(String::length))
```

상수별 클래스 몸체와 데이터를 사용한 열거 타입
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y{
            return x+y;
        }
    },
    MINUS("-"){
        public double apply(double x, double y{
            return x-y;
        }
    },
    TIMES("*"){
        public double apply(double x, double y){
            return x*y;
        }
    }, 
    DIVIDE("/"){
        public double apply(double x, double y){
            return x/y;
        }
    };

    private final String symbol; 
    Operation(String symbol) { 
        this.symbol = symbol;
    }

    @Override 
    public String toString() {
        return symbol; 
    }
    public abstract double apply(double x, double y);
}
```

함수 객체(람다)를 인스턴스 필드에 저장해 상수별 동작을 구현한 열거 타입
```java
public enum Operation {
   
    PLUS("+", (x, y) -> x + y), 
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y), 
    DIVIDE("/", (x, y) -> x / y)
    ;
    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op){
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }
    public double apply(double x, double y){
        return op.applyAsDouble(x, y);
    }
}
```
```
DoubleBinaryOperator는 java.util.function 패키지가 제공하는 다양한 함수 인터페이스 중 하나로 double 타입 결과를 돌려준다
```

**람다는 이름이 없고, 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다**

**람다는 한 줄일 때 가장 좋고, 세 줄 안에 끝내는게 가장 좋다. 세 줄을 넘어가면 가독성이 심하게 나빠진다**

**람다는 함수형 인터페이스에서만 쓰인다**

**람다를 직렬화하는 일은 극히 삼가야 한다(익명 클래스의 인스턴스도 마찬가지이다)**

### 람다가 아닌 익명클래스를 써야하는 경우
1. 추상클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니 익명 클래스를 써야 한다.
예시 
```java
public class Main {

    public static void main(String[] args){

        introduction(new Person(){
            @Override
            String whoAmI() {
                return "My name is Lucy";
            }
        });
    }
    static void introduction(Person p){
        System.out.println(p.whoAmI());
    }
}

abstract class Person {
    abstract String whoAmI();
}
```
--> 적절한 예인지 모르겠지만 추상클래스의 인스턴스를 생성할 때에는 람다식이 불가능하다 이렇게 익명 클래스로 해야한다!

2. 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 쓸 수 있다. 
예시
```java

```
3. 람다는 자신을 참조 할 수 없다( 람다의 this 키워드는 바깥 인스턴스를 가리킨다.) 반면, 익명 클래스에서의 this는 익명 클래스의 인스턴스 자신을 가리킨다. 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.


```
핵심정리 
익명클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들때만 사용하라
```
