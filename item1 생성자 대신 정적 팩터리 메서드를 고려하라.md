# item1. 생성자 대선 정적 메서드를 고려하라 

클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다.

```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

### 정적 팩터리 메서드가 생성자 보다 좋은 장점 5가지.

1. 이름을 가질 수 있다.
   
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 반면 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

예를 들어 

`BigInteger(int, int Random)`와 `BigInteger.probablePrime` 중 어느쪽이 `값이 소수인 BigInteger를 반환하라`는 의미를 잘 설명할 수 있는가. 

   
2. 호출될 때 마다 인스턴스를 새로 생성하지는 않아도 된다. 

덕분에 불변클래스(immutable class)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
`Boolean.valueof(boolean)` 메서드는 객체를 아예 생성하지 않는다. 

3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 

반환할 객체의 클래스를 자유롭게 선택할 수 있는 엄청난 `유연성`을 제공하는데 API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 

```java
public class MyBook{
    public MyBook(){}
    public static MyBook getChildInstance() {
        return MyBookChild.getInstance();
    }
}
public class MyBookChild extends MyBook{
    private MyBookChild(){}
    public static MyBookChild getInstance() {
        return new MyBookChild();
    }
}
```
이런 방법으로 인해, API를 만들 때 이런 유연성을 응용하면 구현 클래스(MyBookChild)를 공개하지 않고도 MyBook을 통해 구현 클래스를 반환할 수 있어 API와 소통이 가능하다. (클라이언트는 상위 클래스의 정적 팩토리만 접근하여 구현 클래스의 메서드를 호출할 수 있는 것이다.)

참조 - <https://it-mesung.tistory.com/184>

4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다. 

반환타입이 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 
```java
 public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
     Enum<?>[] universe = getUniverse(elementType);
     if (universe == null)
        throw new ClassCastException(elementType + " not an enum");
    
    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
    }
```
팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수 없고 알 필요도 없다. EnumSet의 하위 클래스이기만 하면 되는 것이다. 

5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. ex) JDBC

서비스 제공자 프레임 워크는 3개의 핵심 컴포넌트로 이뤄진다. 
   1) 서비스 인터페이스: 구현체의 동작을 정의 ex) `Connection`
   2) 제공자 등록 API: 제공자가 구현체를 등록 ex) `DriverManager.registerDriver`
   3)  서비스 접근 API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용 ex) `DriverManager.getConnection` 
   4)  서비스 제공자 인터페이스: 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명 ex) `Driver`

### 정적 팩터리 메서드의 단점 
1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 

2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. 

정적 팩터리 메서드에 흔히 사용하는 명명 방식.
- `from` : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 
```java 
Date d = Date.from(instant);
```
- `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```
- `valueOf` : from과 of의 더 자세한 버전
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```
- `instance` 혹은 `getInstance` : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.  
```java
StackWalker luke = StackWalker.getInstance(options);
```
- `create` 혹은 `newInstance` : `instance` 혹은 `getInstance`와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다. 
```java
Object newArray = Array.newInstance(classObject, arrayLen);
```
- `getType` : `getInstance` 와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. `Type` 은 팩터리 메서드가 반환할 객체 타입이다. 
```java
FileStore fs = Files.getFileStore(path);
```
- `newType`: `newInstance` 와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. `Type`은 팩터리 메서드가 반환할 객체의 타입이다. 
```java
BufferedReader br = Files.newBufferedReader(path);
```
- `type` : `getType`과 `newType`의 간결한 버전 
```java
List<Complaint> litany = Collections.list(legacyLitany);
```

```
핵심정리 

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으나 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 패터리를 사용하는게 유리한 경우가 더 많으므로 무작정 pulbic 생성자를 제공하던 습관이 있다면 고치자.
```




