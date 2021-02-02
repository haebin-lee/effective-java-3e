# item35. ordinal 메서드 대신 인스턴스 필드를 사용하라

열거타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다

ordinal을 잘 못 사용한 예 - 따라하지 말 것
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```
상수의 선언 순서를 바꾸는 순간 `numberOfMusicians`는 오동작하고, 이미 사용중인 정수와 같은 상수는 추가할 방법이 없다. 절대 이렇게 써선 안된다. **열거타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자.**

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), NONET(9), DECTET(10);

    private final int numberOfMusicians;
    Ensemble(int size) {
        this.numberOfMusicians = size;
    }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

Enum의 API 문서를 보면 ordinal에 대해 이렇게 쓰여 있다. "대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설게 되었다." 따라서 이런 용도가 아니라면 Ordinal 메서드는 절대 사용하지 말자.