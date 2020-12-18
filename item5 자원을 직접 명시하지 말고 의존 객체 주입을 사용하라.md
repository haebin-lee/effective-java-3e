# item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 

정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트 하기 어렵다. 
```java
public class SpellChecker {
    private static final Lexicon dictionary=...;

    private SpellChecker() {} // 객체 생성 방지 

    public static boolean isValid(String word){...}
    public static List<String> suggestions(String typo) {...}
}
```

싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다. 
```java
public class SpellChecker {
    private static final Lexicon dictionary=...;
    
    private SpellChecker() {} // 객체 생성 방지 
    public static SpellChecker INSTANCE = new SpellChecker(...); // 싱글턴 

    public static boolean isValid(String word){...}
    public static List<String> suggestions(String typo) {...}
}

```

final 한정자를 제거 하고 다른 사전으로 교체하는 메서드를 추가할 수 있지만, 아쉽게도 이 방식은 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸 수 없다. 

**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다**

대신 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야하며, 클라이언트가 원하는 자원(dictionary) 을 사용하기 위해서는 **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식으로 쓰자**. 이는 의존 객체 주입의 한 형태로, 맞춤법 검사기를 생성할 때 의존 객체인 사전을 주입해주면 된다. 


의존 객체 주입은 유연성과 테스트의 용이성을 높여준다.
```java
public class SpellChecker{
    private final Lexicon dictionary;

    // 의존 객체 주입 방식
    public SpellChecker(Lexicon dictionary){
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public static boolean isValid(String word){...}
    public static List<String> suggestions(String typo) {...}
}
```

생성자에 자원 팩터리를 넘겨주는 방식이 있다. 팩터리란 호출할 때 마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다. 
```java
Supplier<T>
```

```
핵심정리

클래스가 내부적으로 하나 이상이 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을(혹은 그 자원을 만들어주는 팩터리를) 생성자에(혹은 정적 패터리나 빌더에) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.  
```