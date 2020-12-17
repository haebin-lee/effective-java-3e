# item3. private 생성자나 열거타입으로 싱글턴임을 보증하라 

싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 

public static final 필드 방식의 싱글턴 
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { .. }

    public void leaveTheBuilding() { ... }
}
```
장점 
1. 해당 클래스가 싱글턴인 것이 명백히 드러난다. `public static` 필드가 `final`이니 절대로 다른 객체를 참조할 수 없다.
2. 간결함

정적팩터리 방식의 싱글턴 
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { .. }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilder() { ... }
}
```
1. API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다
2. 정적 팩터리를 제너릭 싱글턴 팩터리로 만들수 있다.(아이템30)
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다. `Elvis::getInstance` 를 `Supplier<Elvis>`로 사용하는 형태이다. 

이런 장점들이 굳이 필요하지 않으면 public 필드 방식이 좋다. 

열거타입 방식의 싱글턴 - 바람직한 방법 
```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilder() {...}
}
```
대부분 상황에서는 원소가 하나뿐인 열거타입이 싱글턴을 만드는 가장 방법이다. 하지만, 만들려는 싱글턴이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다 

