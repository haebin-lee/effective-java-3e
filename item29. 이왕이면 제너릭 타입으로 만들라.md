# item29. 이왕이면 제너릭 타입으로 만들라 

Object 기반 스택 - 제너릭이 절실한 강력 후보.
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if(size ==0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
클라이언트는 스택에서 꺼낸 객체를 형변환해야하는데, 이때 런타임 오류가 날 위험이 있다.

```java
public class Stack<E> {

    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        // 여기서 문제가 발생함
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

}
```

문제 해결 방법 
1. 제너릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다. Object 배열을 생성한 다음 제너릭 배열로 형변환해보자. 
```java
 public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
```
- 프로그램의 안정성을 해치지 않음을 우리 스스로 확인해야 한다. 
- 비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 `@SuppressWarnings` 애너테이션으로 해당 경고를 숨긴다. 

```java
// 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다. 
    // 따라서 타입 안정성을 보장하지만, 
    // 이 배열의 런타임 타입은 E[]가 아닌 Object[]다.
    @SuppressWarnings("unchecked")
    public Stack2() {
        elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];
    }
```

2. elements의 필드의 타입을 `E[]`에서 `Object[]`로 바꾸는 것이다. 
```java
public class Stack<E> {

    private Object[] elements; // 변경 
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        @SuppressWarnings("unchecked") E result = (E) elements[--size]; // 변경
        // push에서 E타입만 허용하므로 이 형변환은 안전하다.
        elements[size] = null;
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

첫번째 방법은 가독성이 더 좋다. 배열의 타입을 `E[]`로 선언하여 오직 E타입 인스턴스만 받음을 확실히 어필한다. 코드도 더 짧다. 

두번째 방식에서는 배열에서 원소를 읽을 때 마다 해줘야 한다. 따라서 현업에서는 첫번재 방식을 더 선호하며 자주 사용한다. 


Stack에서 꺼낸 원소에서 String의 toUpperCase 메서드를 호출할 때 명시적 형변환을 수행하지 않으며 (컴파일러에 의해 자동 생성된) 이 형변환이 항상 성공함을 보장한다 
```java
Stack2<String> stack2 = new Stack2<>();
for(String arg: test)
    stack2.push(arg);
while(!stack2.isEmpty()){
    System.out.println(stack2.pop().toUpperCase());
}
```

```
핵심정리 

클라이언트에서 직접 형변환해야 하는 타입보다 제너릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. 그렇게 하려면 제너릭 타입으로 만들어야 하는 경우가 많다. 기존 타입 중 제너릭이었어야 하는 게 있다면 제너릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서 새로운 사용자를 훨신 편하게 해주는 길이다.
```