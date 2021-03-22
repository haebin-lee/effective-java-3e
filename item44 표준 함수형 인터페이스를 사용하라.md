# item44. 표준 함수형 인터페이스를 사용하라.

상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴보다 현대적인 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다. 

**필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.**

| 인터페이스 | 함수 시그니처 | 예|
|---------|------------|--|
|`UnaryOperator<T>` | T apply(T t) | String::toLowerCase |
|`BinaryOperator<T>`| T apply(T t1, T t2) | BigInteger::add |
|`Predicate<T>` | boolean test(T t) | Collection::isEmpty |
|`Function<T,R>` | R apply(T t) | Arrays::asList |
|`Supplier<T>` | T get() | Instant::now |
|`Consumer<T>` | void accept(T t) | System.out::println |

다음 조건을 만족한다면 전용 함수형 인터페이스를 구현해야 하는 건 아닌지 검토하자.
1. 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다. 
2. 반드시 따라야 하는 규약이 있다. 
3. 유용한 디폴트 메서드를 제공할 수 있다. 

직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하라.

목적은 세 가지이다.
1. 그 인터페이스가 람다용으로 설꼐 된 것임을 알려준다. 
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일 되게 해준다. 
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다. 
