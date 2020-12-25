# item26. 로 타입은 사용하지 말라

클래스와 인터페이스 선언에 타입 매개변수가 쓰이면 이를 `제너릭 클래스` 혹은 `제너릭 인터페이스`라고 한다. 제너릭 클래스와 제너릭 인터페이스를 통틀어 `제너릭 타입` 이라고 한다.

- `매개변수화 타입(parameterized type)` ex) List\<String>은 원소타입이 String인 리스트를 뜻하는 매개변수화 타입이다. 
- `로 타입(raw type)` ex) List

컬렉션의 로타입 - 따라하지 말 것!
```java
//Stamp 인스턴스만 취급한다. 
private final Collection stamps = ...;

// 실수로 도장(stamp) 대신 동전(coin)을 넣어도 아무 오류 없이 컴파일이 되고 실행된다. 
stamps.add(new Coin(...))
```

반복자의 로 타입 - 따라하지 말 것!
```java
for(Iterator i=stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next(); // ClassCastException이 발생된다.
    stamp.cancel();
}
```
제너릭을 활용하면 이런 정보가 주석이 아닌 타입 선언 자체에 녹아든다. 

매개변수화 된 컬렉션 타입 - 타입 안정성 확보!
```java
private final Collection<Stamp> stamps = ...;
```

**로 타입을 쓰면 제너릭이 안겨주는 안정성과 표현력을 모두 잃게 된다.**

List와 같은 로 타입은 사용해서는 안되나 List<Object> 처럼 임의의 객체를 허용하는 매개변수화 타입은 괜찮다. 
- List : 제너릭 타입에서 완전히 발 뺌(책의 표현)
- List\<Object> : 모든 타입을 허용한다는 의미

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
    System.out.println(s);
}

private static void unsafeAdd(List list, Object o){
    list.add(o);
}
```
이 코드는 컴파일은 되지만 로 타입인 List를 사용하여 다음과 같은 경고가 발생한다. 
```
Exception in thread "main" java.lang.ClassCastException: java.base/java.lang.Integer cannot be cast to java.base/java.lang.String
	at com.lucy.generic.Main.main(Main.java:11)
```

`List`를 `List<Object>` 로 변경하면 아예 컴파일 조자 되지 않는다. 
```
java: incompatible types: java.util.List<java.lang.String> cannot be converted to java.util.List<java.lang.Object>
```

잘못된 예 - 모르는 타입의 원소도 받는 로 타입을 사용했다. 
```java
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0; 
    for (Object o1 : s1) {
        if (s2.contains(o1)) {
            result ++;
        }
    }
    return result;
}
```
위 메서드는 동작하지만 로 타입을 사용해 안전하지 않다. 따라서 `비한정적 와일드카드 타입(unbounded wildcard type)`을 대신 사용하는게 좋다
ex) `Set<E>` -> `Set<?>` : 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set타입이다. 

비한정적 와일드 카드 타입을 사용하라 - 타입 안전하며 유연하다. 
```java
static int numElementsInCommon(Set<?> s1, Set<?> s2){...}
```

`Set<?>`와 `Set`의 차이는 뭘까? 와이들 타입은 안전하고, 로 타입은 안전하지 않다. (반면 Collection<?>에는 null 외에는 어떤 원소도 넣을 수 없다.)

추가로 
**class 리터럴에는 로 타입을 써야 한다.** 예를 들어 List.cass, String[].class, int.class는 허용하고, List\<String>.class와 List\<?>.class 는 허용하지 않는다.


**`instanceof` 연산자는 로 타입이든 비한정적 와일드 카드 타입이든 완전히 동일하게 동작하므로 차라리 로 타입을 쓰는 편이 깔끔하다.**

로 타입으로 써도 좋은 예 - instanceof 연산자 
if (o instanceof Set) {
    Set<?> s = (Set<?>) o; 
}
o의 타입이 Set임을 확인하고 다음 와일드 카드 타입인 Set<?>로 형변환해야 한다(로타입인 Set이 아니다).


```
로 타입을 사용하면 런타임에 예외가 일어날 수 있으니 사용하면 안 된다. 로 타입은 제너릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다.

Set<Object> 는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set<?>는 모종의 타입 객체만 저장할 수 있는 와일드카드 타입이다. 그리고 이 로 타입인 Set은 제너릭 타입 시스템에 속하지 않는다. Set<Object> 와 Set<?> 는 안전하지만, 로 타입인 Set은 안전하지 않다.
```

| 한글용어 | 영문 용어 | 예 | 아이템 |
|:-------|:-------|:---|:-----|
|매개변수화 타입 | parameterized type | List\<String> | 아이템 26|
|실제 타입 매개변수 | actual type parameter | String | 아이템 26 |
|제너릭 타입 | generic type | List\<E> | 아이템 26, 29|
|정규 타입 매개변수 | formal type parameter | E | 아이템 26 |
|비한정적 와일드 카드 타입 | unbounded wildcard type | List<?> | 아이템 26|
|로 타입 | raw type | List | 아이템 26|
|한정적 타입 매개변수 | bounded type parameter | \<E extends Number> | 아이템 29|
|재귀적 타입 매개변수 | recursive type bound | \<T extends Comparable\<T>> |아이템 30|
|한정적 와일드 카드 타입 | bounded wildcard type | List\<? extends Number> | 아이템 31|
|제너릭 메서드 | generic method | static \<E> List\<E> asList(E[] a) | 아이템 30 |
|타입 토큰 | type token | String.class | 아이템 33 | 

