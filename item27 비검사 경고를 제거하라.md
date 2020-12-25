# item27. 비검사 경고를 제거하라. 

대부분 비검사 경고는 쉽게 제거 할 수 있다. 
```java
Set<Lark> exaltation = new HashSet();
```
사실 컴파일러가 알려준 타입 매개변수를 명시하지 않고 자바 7부터 지원하는 다이아몬드 연산자(<>)만으로 해결할 수 있다. 
그러면 컴파일러가 올바른 실제 타입 매개변수 (이 경우는 Lark)를 추론해준다.
```java
Set<Lark> exaltation = new HashSet<>();
```

**할 수 있는 한 모든 비검사 경고를 제거하라 모두 제거한다면 그 코드는 타입 안정성이 보장된다.**

**경고를 제거할수는 없지만 타입이 안전하다고 확신할 수 있다면 `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자**


**`@SuppressWarnings("unchecked")`은 항상 가능한 좁은 범위에 적용하자.** 보통은 변수선언, 아주 짧은 메서드, 혹은 생성자가 될 것이다. 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안된다. 

```java
public <T> T[] toArray(T[] a) {
        if(a.length < size) 
            return (T[]) Arrays.copyOf(elements, size, a.getClass());
        System.arraycopy(elements, 0, a, 0, size);
        if(a.length > size) 
            a[size] = null; 
        return a;
    }
```
return 문에서 비검사 경고가 발생하는데 return문에는 애너테이션을 추가할 수 없다. 

지역변수를 추가해 @SuppressWarnings의 범위를 좁힌다.
```java
public <T> T[] toArray(T[] a) {
        if(a.length < size) {
            @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
            return result;
        }
        System.arraycopy(elements, 0, a, 0, size);
        if(a.length > size)
            a[size] = null;
        return a;
    }
```

@SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다. 다른 사람이 그 코드를 이해하는데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘 못 수정하여 타입 안정성을 잃는 상황을 줄여준다. 

```
핵심정리 

비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최서능ㄹ 다해 제거하라. 경고를 없앨 방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라. 그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라.
```