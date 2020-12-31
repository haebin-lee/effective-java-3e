# item21. 인터페이스는 구현하는 쪽으로 생각해 설계하라. 

자바 8 이전에는 기존 구현체를 깨지 않고는 인터페이스에 메서드를 추가할 방법이 없었다. 자바 8에 와서야 기존 인터페이스에 메서드를 추가할 수 있도록 `디폴트 메서드`를 소개했지만, 위험이 완전히 사라진 것은 아니다. 

`디폴트 메서드`를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의 하지 않은 모든ㄷ 클래스에서 디폴트 구현이 쓰이게 된다. 

자바 8에서는 람다를 활용하기 위해 핵심 컬렉션 인터페이스에 다수의 디폴트 메서드가 추가되었다. 

디폴트 메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동하지만 **생각할수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어렵다.**

`removeIf` 메서드는 `불리언 함수(predicate)가 true를 반환하는 모든 원소를 제거한다.`

자바 8의 collection 인터페이스에 추가된 디폴트 메서드 
```java
default boolean removeIf(Predicate<? super E> filter) {
    Object.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator() ; it.hasNext() ; ) {
        if(filter.test(it.next())) {
            it.remove(); 
            result = true;
        }
    }
    return result;
}
```

`SysnchronizedCollection`는 인스턴스를 여러 스레드가 공유하는 환경에서 한 스레드가 `removeIf`를 호출하면 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다. 