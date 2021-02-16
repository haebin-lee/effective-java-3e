# item63. 문자열 연결은 느리니 주의하라

**문자열 연결 연산자로 문자열 n개를 잇는 시간은 n<sup>2</sup>에 비례한다**

문자열 연결을 잘못 사용한 예 - 느리다. 
```java
public String statement() {
    String result = "";
    for (int i = 0 ; i < numItems() ; i++) 
        result += lineForItem(i);
    return result;
}
```

품목이 많을 때에는 심각하게 느려질 수 있다. 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자.

StringBuilder를 사용하면 문자열 연결 성능이 크게 개선된다.
```java
public String statement2() {
    StringBulider b = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0 ; i < numItems() ; i++) 
        b.append(lineForItem(i));
    return b.toString();
}
```
statement 메서드의 수행 시간은 품목 수의 제곱이 비례해 늘어나고 statement2는 선형으로 늘어난다. 

**많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자**