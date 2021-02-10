# item61. 박싱된 기본 타입보다는 기본 타입을 사용하라.

기본 타입: `int`, `double`, `boolean`

참조 타입: `String`, `List`

기본타입의 박싱된 기본타입 : `Integer`, `Double`, `Boolean`


기본타입과 박싱 타입의 차이 

| 기본 타입 | 박싱된 기본 타입|
|---------|-------------|
|값 | 값 + 식별성(identity) : 값이 같아도 서로 다르다고 식별가능|
|항상 유효 | null을 허용 |
|시간과 메모리 사용면에서 더 효율적 | |

잘못 구현된 비교자 - 문제를 찾아보자!
```java
Comparator<Intege>r naturalOrder = (i, j) = (i < j) ? -1 : (i == j ? 0 : 1);
```

`naturalOrder.compare(new Integer(42), new Integer(42))`의 경우 문제가 생긴다. 

i와 j가 서로 다른 Integer 인스턴스라면 (비록 값은 같더라도) 이 비교으 결과는 false가 되고, 비교자는 (잘못된 결과인) 1을 반환한다. 이처럼 (같은 객체를 비교하는게 아니라면) 박싱된 기본 타입에 == 연산자를 사용하면 오류가 일어난다. 

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, j = jBoxed; 
    return i < j ? -1 : (i ==j ? 0 : 1);
};
```

기이하게 동작하는 프로그램 - 결과를 맞춰보자 
```java
public class Unbelievable {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42) {
            System.out.println("믿을 수 없군!");
        }
    }
}
```
- i == 42를 검사할 때 NullPointerException을 발생시킨다. 
- **거의 예외 없이 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다.**
- 간단히 i를 int로 선언해주면 된다. 

```java
public static void main(String[] args) {
    Long sum = 0L;
    for(long i = 0 ; i <= Integer.MAX_VALUE ; i++) {
        sum += i; 
    }
    System.out.println(sum);
}
```
- 오류나 경고 없이 컴파일 되지만, 박싱과 언박싱이 반복해서 일어나 체감될 정도로 성능이 느려진다. 

박싱된 기본 타입은 언제써야 할까?
- 컬렉션 원소, 키, 값으로 쓴다. 
- 매개변수화 메서드의 타입 매개변수로 사용한다. ex) `ThreadLocal<Integer>`
- 리플렉션을 통해 메서드를 호추할 때 사용한다.