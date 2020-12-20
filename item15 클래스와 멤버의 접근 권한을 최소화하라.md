# item15. 클래스와 멤버의 접근 권한을 최소화하라 

어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 **바로 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐다.**

> 정보은닉의 장점 
1. 시스템 개발 속도를 높인다. 
2. 시스템 관리 비용을 낮춘다. 
3. 정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다. 
4. 소프트웨어 재사용성을 높인다. 
5. 큰 시스템을 제작하는 난이도를 낮춰준다. 

기본원칙은 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다. 

패키지 외부에서 쓸 이유가 없다면 `package-private`으로 선언하자 

`public`으로 선언하면 API가 되므로 하위 호환을 위해 영원히 관리해줘야만 한다. 

`public`일 필요가 없는 클래스의 접근 수준을 `package-private` 톱 클래스로 좁히는 일이다. 

| 접근  | 설명 |
|----------|---------------------------------------|
|`private` | 멤버를 선언한 톱 레벨 클래스에서만 접근할 수 있다.|
|`package-private` | 멤버가 소속된 패키지 안의 모든 클래스에서 접근 할 수 있다. 접근 제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준이다. (단, 인터페이스의 멤버는 기본적으로 public이 적용된다)|
|`protected` | package-private의 접근 범위를 포함하며, 이 멤버를 선언한 클래스의 하위 클래스에서도 접근할 수 있다. |
|`public`| 모든 곳에서 접근할 수 있다. |

`protected` 멤버의 수는 적을수록 좋다. 

상위 클래스의 메서드를 재정의할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다는 것이다.

`public` 클래스의 인스턴스 필드는 되도록 `public` 이 아니어야 한다. `public` 가변 필드를 갖는 클래스는 일반적으로 스레드 안전하지 않다. 

클래스에서 `public static final` 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다. 

```java
// 보안 허점이 숨어 있다. 
public static final Things[] VALUES = {...};
```

해결책1) 배열을 private으로 만들고 public 불변 리스트룰 추가한다.
```java
private static final Thing[] PRIVATE_VALUES ={...};
public static final List<Thing> VALUES = Collections.unmodifiableList(ARrays.asList(PRIVATE_VALUES));
```

gorufcor 2) 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법이다. 
```java
private static final Thing[] PRIVATE_VALUES = {...};
publci static final Thing[] values () {
    return PRIVATE_VALUES.clone();
}
```


```
핵심 정리 
프로그램 요소의 접근성을 가능한 한 최소한으로 하라. 꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외에는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개되는 일이 없도록 해야한다. public 클래스는 상수용 public static final 필드 외에는 어떠한 필드고 가져서는 안된다. public static final 필드가 참조하는 객체가 불변인지 확인하라.
```
