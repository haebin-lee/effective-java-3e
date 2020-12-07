# item22. 인터페이스는 타입을 정의하는 용도로만 사용하라 

클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기해주는 것이다. 


> 잘 못 사용한 예
```java
public interface PhysicalConstants {
    // 아보가드로 수 
    static final double AVOGADROS_NUMBER=6.022_140_857e23;
    // 볼츠만 상수 
    static final double BOLTZMANN_CONSTANT = 1.380__648_52e-23;
    // 전자 질량
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```
1. 클래스가 어떤 상수 인터페이스를 사용하든 사용자 에게는 아무런 의미가 없다. 
2. 오히려 사용자에게 혼란을 주기도 하며, 
3. 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 한다.
4. `final` 이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름 공간이 그 인터페이스가 정의한 상수들로 오염이 된다. 

> 상수 유틸리티 클래스 
```java
public class PhysicalConstants {
    private PhysicalConstants() {} // 인스턴스화 방지

    // 아보가드로 수 
    public static final double AVOGADROS_NUMBER=6.022_140_857e23;
    // 볼츠만 상수 
    public static final double BOLTZMANN_CONSTANT = 1.380__648_52e-23;
    // 전자 질량
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

```
핵심 정리
인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.
```