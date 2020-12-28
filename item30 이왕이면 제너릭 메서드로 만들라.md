# item30 이왕이면 제너릭 메서드로 만들라. 

클래스와 마찬가지로 메서드도 제너릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제너릭이다. 예를 들어 `Collections`의 `binarySearch`, `sort` 등이 있다.

로 타입으로 사용 - 수용 불가 
```java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);
    return result;
}
```

컴파일은 되지만, 경고가 발생한다. 

제너릭 메서드 
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

제너릭 메서드를 활용하는 간단한 프로그램 
```java
public static void main(String[] args) {
    Set<String> guys = Set.of("톰", "딕", "해리"); 
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```

위를 한정적 와일드카드 타입을 사용하여 더 유연하게 개선할 수 있다. 

제너릭은 런타임에 타입 정보가 소거 되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개 변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 


항등함수를 만들어보자 

제너릭 싱글턴 팩터리 패턴
```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}`
```
T가 어떤 타입이든 `UnaryOperator<Object>`는 `UnaryOperator<T>`가 아니지만, 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별 함수이므로, T가 어떤 타입이든 `UnaryOperator\<T>`를 사용해도 타입 안전하다. `@SuppressWarnings` 애너테이션을 추가하면 오류나 경고 없이 컴파일 된다. 

제너릭 싱글턴을 사용하는 예
```java
public static void main(String[] args) {
    String[] strings = {"삼베", "대마", "나일론"};
    UnaryOperator<String> sameString  = identityFunction();
    for (String s: strings) {
        System.out.println(sameString.apply(s));
    }

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = identityFunction();
    for (Number n : numbers) {
        System.out.println(sameNumber.apply(n));
    }
}
```

자기자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 바로 `재귀적 타입 한정(recursive type bound)`라는 개념이다.

```java
public interface Comparable<T> {
    int compareTo(T o);
}
```
String은 `Comparable<String>`을 구현하고 Integer은 `Comparable<Integer>`을 구현하는 식이다. 

재귀적 타입 한정을 이용해 상호비교할 수 있음을 표현했다. 
```java
public <E extends Comparable<E>> E max(Collection<E> c);
```

`<E extends Comparable<E>>`는 `모든 타입 E는 자신과 비교할 수 있다.`라고 읽을 수 있다. 상호 비교 가능하다는 뜻을 아주 정확하게 표현했다고 할 수 있다.

컬렉션에서 최대값을 반환한다. - 재귀적 타입 한정 사용
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
    }

    E result = null;
    for (E e: c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```

```
핵심정리 

제너릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환 값을 명시적으로 형변환 해야하는 메서드보다 제너릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제너릭 메서드가 되어야 한다.
```