# item62. 다른 타입이 적절하다면 문자열 사용을 피하라

문자열은 다른 값 타입을 대신하기에 적합하지 않다.
- 받은 데이터가 수치형이라면 ㅑnt, float, BigInteger 등
- 예/아니오 질문의 답이라면 열거형 타입이나 Boolean

문자열은 열거 타입을 대신하기에 적합하지 않다. 
- 상수를 열거할 때는 문자열보다 열거 타입이 월등이 낫다.

문자열은 혼합 타입을 대신하기에 적합하지 않다. 

혼합타입을 문자열로 처리한 부적절한 예 
```java
String compoundKey = className + "#" + i.next();
```
- 문자 #이 두 요소 중 하나에서 스였다면 혼란스러운 결과를 초래한다. 
- 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다. 

문자열은 권한을 표현하기에 적합하지 않다.

잘못된 예 - 문자열을 사용해 권한을 구분하였다. 
```java
public class ThreadLocal {
    private ThreadLocal() {} // 객체 생성 불가

    // 현 스레드의 갑승ㄹ 키로 구분해 저장한다. 
    public static void set(String key, Object value); 

    // (키가 가리키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```
- 스레드 구분용 문자열 키가 전역 이름 공간에 공유되어 같은 키를 쓰기로 결정한다면 의도치 않게 같은 변수를 공유하게 된다. 

Key 클래스로 권한을 구분했다.
```java
public class ThreadLocal {
    private ThreadLocal() {} // 객체 생성 불가 
    
    public static class Key {
        Key() {}
    }
    public static Key getKey() {
        return new Key();
    }
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

리팩터링하여 Key를 ThreadLocal로 변경
```java
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

매개변수화하여 타입안정성을 확보 
```java
public final class ThreadLocal<T> {
    public ThreadLocal(); 
    public void set(T value);
    public T get();
}
```


