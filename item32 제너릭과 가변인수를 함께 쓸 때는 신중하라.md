# item 32. 제너릭과 가변인수를 함께 쓸 때는 신중하라. 

`가변인수는 메서드에 넘기는 인수의 갯수를 클라이언트가 조절할 수 있게` 해주는데, 구현방식에 허점이 있다. 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다. 그런데 내부로 감춰야 했을 이 배열을 그만 클라이언트에 노출하는 문제가 생겼다. 그 결과 varags 매개변수에 제너릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다. 


매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.
```
warning: [unchecked] Possible heap pollution from parameterized vararg type List<String>
```

제너릭과 varargs를 혼용하면 타입 안정성이 깨진다. 
```java
static void dangerous(List<String>... stringList) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringList;
    objects[0] = intList; // 힙 오염 발생
    String s = stringList[0].get(0); // ClassCastException
}
```
타입 안정싱이 깨지므로 `제너릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.`

실무에서 유용하기 때문에 제너릭이나 매개변수화 타입의 varargs 매개변수를 받도록 설계되었고, 자바7 이전에는 제너릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 할 수 있는 일이 없었지만, 자바 7에서는 `@SafeVarargs` 애너테이션이 추가되어 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다. 
`@SafeVarargs 애너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이다.`

`@SafeVarargs`는 메서드가 안전할때만 붙일 수 있는데 
1. varargs 매개변수를 담는 제너릭 배열에 아무것도 저장하지 않고, 참조가 밖으로 노출되지 않는다고 신뢰할 때 타입안전하다. 
2. varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면(varargs의 목적대로 쓰인다면) 그 메서드는 안전하다. 

자신의 제너릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다.
```java
static <T> T[] toArray(T...args) {
    return args;
}
```

```java
static <T> T[] pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError(); // 도달할 수 없다.
}
```
pickTwo는 반환값을 attributes에 저장하기 위해 String[]로 형변환하는 코드를 컴파일러가 자동 생성하고, Object[]는 String[]의 하위 타입이 아니므로 이 형변환은 항상 실패한다. 

제너릭 varargs 매개변수를 안전하게 사용하는 메서드
```java
@SafeVarargs
static <T> List<T> flatten(List<? extends  T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

제너릭이나 매개변수화 타입이 varargs 매개변수를 받는 모든 메서드에 `@SafeVarargs`를 달아라

둘 중 하나라도 어겼다면 수정하라!
- varargs 매개변수 배열에 아무것도 저장하지 않는다. 
- 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

```html
@SafeVarargs 애너테이션은 재정의할 수 없는 메서드에만 달아야 한다. 재정의한 메서드도 안전할지는 보장할 수 없기 때문이다. 자바 8에서는 이 애너테이션은 오직 정적 메서드와 final 인스턴스 메서드에만 붙일 수 있고, 자바 9부터는 private 인스턴스 메서드에도 허용된다.
```

제너릭 varargs 매개변수를 List로 대체한 예 - 타입안전하다.
```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```
`List.of`에 `@SafeVarargs` 애너테이션이 있기 때문에 안전하다.

```java
static <T> List<T> pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
        case 0: return List.of(a, b);
        case 1: return List.of(a, c);
        case 2: return List.of(b, c);
    }
    throw new AssertionError(); // 도달할 수 없다.
}
```

```html
가변인수와 제너릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제너릭의 타입규칙이 서로 다르기 때문이다. 제너릭 varargs 매개변수는 타입 안전하지는 않지만 허용된다. 메서드에 제너릭 (혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는데 불편함이 없게끔 하자.
```