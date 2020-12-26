# item28. 배열보다는 리스트를 사용하라. 

배열과 제너릭 타입에는 중요한 차이가 두 가지가 있다. 
1. 배열은 공변(covariant) 이다. 
- 공변이란 sup가 super의 하위타입이라면 sup[]는 super[]의 하위타입이 된다. (공변; 함께 변한다는 뜻이다.)
2. 제너릭은 불공변(invariant) 이다. 
- List\<Type1>은 List\<Type2>의 하위 타입도 아니고 상위타입도 아니다.

런타임에 실패한다. 
```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라서 넣을 수 없다."; // ArrayStoreException이 발생한다.
```

컴파일 되지 않는다.
```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다.");
```
3. 배열에서는 그 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일 할 때 바로 알 수 있게 된다. 
4. 배열은 실체화(reify) 된다. 
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 하지만 제너릭은 타입 정보가 런타임에는 소거(erasure) 된다. 

배열과 제너릭은 잘 어우러지지 못하는데 `new List<E>[]`, `new List<String[]`, `new E[]` 식으로 작성하면 컴파일 할 때 제너릭 배열 생성 오류를 일으킨다. 

제너릭 배열을 만들지 못하게 막은 이유는 무엇일까? 타입 안전하지 않기 때문이다. 

제너릭 배열 생성을 허용하지 않는 경우 - 컴파일 되지 않는다.
```java
List<String>[] stringLists = new List<String>[1];   // 1
List<Integer> intList = List.of(42);                // 2
Object[] objects = stringLists;                     // 3
objects[0] = intList;                               // 4
String s = stringLists[0].get(0);                   // 5
```
- 제너릭 배열을 생성하는 1이 허용 
- 2,3,4번이 문제 없이 허용되지만, `List<String>` 인스턴스만 담겠다고 선언한 stringLists에 배열에는 지금 List<Integer> 인스턴스가 저장되어 있고, 5에서는 런타임에 `ClassCastException`이 발생하게 된다. 이런 일을 방지하려면 (제너릭 배열이 생성되지 않도록) 1에서 컴파일 오류를 내야 한다. 

배열로 형변환 할 때 제너릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션인 List\<E>를 사용하면 해결된다. 

Chooser- 제너릭을 시급히 적용해야 한다. 
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current(); 
        return choiceArray[rnd.nextInt(choiceArray.length)]
    }
}
```
choose 메서드를 호출할 때 마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 

```java
public class Chooser<T> {
    private final List<T> choiceList; 

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current(); 
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```
코드양이 좀 더 늘었고, 조금 더 느리지만 런타임에 ClassCastException을 만날 일은 없으니 그만한 가치가 있다. 

```
핵심정리 

배열과 제너릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화되는 반면, 제너릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지만 컴파일 타임에는 그렇지 않다. 제너릭은 반대다. 그래서 둘은 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자
```
