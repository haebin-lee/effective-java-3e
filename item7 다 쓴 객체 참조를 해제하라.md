# item7. 다 쓴 객체 참조를 해제하라 

```java
public class Stack {
    private Object[] elements;
    private int size = 0; 
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if(size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    /*
     * 원소를 위한 공간을 적어도 하나 이상 확보한다. 
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
    */
    private void ensureCapacity(){
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size +1);
    }
}
```
이 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 

가비지 컬렉션 언어에서는 (의도치 않게 객체를 살려두는) 메모리 누수를 찾기가 아주 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체..)를 회수해가지 못한다. 

해당 참조를 다 썼을 때 `null`처리(참조 해제) 하면 된다. 

더이상 필요 없어지는 시점은 스택에서 꺼내질 때다. 
```java
public Object pop() {
    if (size ==0) 
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

하지만 `null` 처리하는 일은 예외적인 경우여야 한다.

메모리 누수를 일으키는 원인 
1. 자기 메모리를 직접 관리하는 클래스 ex) Stack
2. 캐시
- 캐시 외부에서 키(key)를 참조하는 동안만(값이 아니다) 엔트리가 살아있는 캐시가 필요한 상황이라면 `WeakHashMap`을 사용해 캐시를 만들자. 다 쓴 엔트리는 그 즉시 자동으로 제거 될 것이다. 
3. 리스너(listener) 콜백(callback) 

```
핵심정리 
메모리 누수는 겉으로는 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되고 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
```