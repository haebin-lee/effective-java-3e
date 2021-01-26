# item31. 한정적 와일드카드를 사용해 API 유연성을 높이라. 

Stack의 public API 예제 
```java
publlic class Stack<E> {
    public Stack(); 
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```
와일드카드 타입을 사용하지 않는 pushAll 메서드 - 결함이 있다.
```java
public void pushAll(Iterable<E> src) {
    for (E e: src) {
        push(e);
    }
}
```
Iterable src의 원소타입이 스택의 원소타입과 일치하면 잘 작동한다. 하지만 `Stack<Number>`로 선언한 후 `pushAll(intVal)`을 호출하면 어떻게 될까? (intVal=Integer타입) `Integer`는 `Number`의 하위타입이니 잘 동작해야만 할 것 같지만 매개변수화 타입이 불공변이기 때문에 잘 동작하지 않는다. 

한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다. 
E의 Iterable이 아니라 `E의 하위 타입의 Iterable`이어야 하며, 와일드 타입 `Iterable<? extends E>`가 정확히 그런 뜻이다.

생산자(producer) 매개변수에 와일드 카드 타입 적용
```java
public void pushAll(Iterable<? extends E> src) {
    for(E e: src) {
        push(e);
    }
}
```

와일드 카드 타입을 사용하지 않는 popAll 메서드 - 결함이 있다.
```java
public void popAll(Collection<E> dst) {
    while(!isEmpty()) 
        dst.add(pop())
}
```
이 경우는 `E의 상위타입의 Collection`이어야 한다(모든 타입은 자기 자신의 상위 타입이다.) 와일드 카드 타입을 사용한 `Collection<? super E>` 가 정확히 그런 의미이다.

**유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.** 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 수행한다면 와일드 카드 타입을 써도 좋을게 없다.

`팩스(PECS) : producer-extends, consumer-super`

매개변수화 타입 T가 `생산자라면 <? extends T>`를 사용하고, `소비자라면 <? super T>`를 사용하라. Stack 예에서 pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 src의 적절한 타입은 Iterable<? extends E> 이다. 한편 popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입은 Collection<? super E> 이다.

union 메서드를 보자 
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

```java
public static <E> Set<E> union (Set<? extends E> s1, Set<? extends E> s2)
```
반환타입은 여전히 `Set<E>`이다. 반환타입에는 한정적 와일드카드 타입을 사용하면 안된다. 유연성을 높여주기는 커녕 클라이언트 코드에서도 와일드 카드 타입을 써야하기 때문이다. 

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

위의 코드는 자바8부터 가능하고 이전에는 
```java
Set<Number> numbers = Union.<Number> union(integers, doubles);
```
컴파일러가 올바른 타입을 추론하지 못하므로 타입을 명시해줘야 한다. 

```
매개변수(parameter)와 인수(argument)의 차이를 알아보자. 매개변수는 메서드 선언에 정의한 변수이고, 인수는 메서드 호출시 넘기는 '실제 값'이다. 

void add (int value) {...}
add(10)

value는 매개변수이고, 10은 인수이다. 

class Set<T> {...}
Set<Integer> = ...;

T는 매개변수이고, Integer는 타입인수가 된다.
```

```java
public static <E extends Comparable<E>> E max (List<E> list)
```

와일드 타입을 사용하여 위 코드를 수정하면 
```java
public static <E extends Comparable<? super E>> E max (List<? extends E> list)
```

입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? extends E>`로 수정했다. 

`Comparable<E>`는 E 인스턴스를 소비한다(그리고 선후 관계를 뜻하는 정수를 생산한다) 그러므로 `Comparable<E>`로 대체했다. 

swap 메서드의 두가지 선언 
```java
public static <E> void swap (List<E> list, int i, int j); 
public static void swap (List<?> list, int i, int j);
```

public API라면 두번째가 낫다. 기본 규칙은 `메서드 선언에 타입 매개변수가 한번만 나오면 와일드 카드로 대체하라`

하지만 두번째 코드는 컴파일이 안되는데 `List<?>`는 null 외에는 어떤 값도 넣을 수 없다. 그래서 와일드 카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용해야 한다. 

```java
public static void swap (List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드 카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드 
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)))
}
```

`swapHelper`는 리스트가 List<E>임을 알고 있으므로 꺼낸 값이 항상 E이고, E타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다. 


```
조금 복잡하더라도 와일드 카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리르 작성한다면 반드시 와일드 카드 타입을 적절히 사용해줘야한다. PECS 공식을 기억하자. 즉, 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.
```
