# item25. 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱 레벨 클래스를 여러개 선언해도 문제는 없으나, 중복된 클래스가 있을 경우, 어느 소스 파일을 먼저 하는냐에 다라 결과가 달라지게 된다 

### 문제점

Main.java
```java
public class Main {
    public static void main(String[] args){
        System.out.println(Utensil.NAME + Desert.NAME);
    }
}
```

Utensil.java
```java
class Utensil {
    static final String NAME = "pan";
}
class Dessert {
    static final String NAME = "cake";
}
```

Dessert.java
```java
class Utensil {
    static final String NAME = "pot";
}
class Desert {
    static final String NAME = "pie";
}
```


```bash
javac Main.java Dessert.java
```
> 컴파일 오류가 나면서, Utensil과 Dessert 클래스를 중복 정의했다고 알려줄 것이다. 
>> 1) `Main.java`를 먼저 컴파일 
>> 2) (Dessert 참조보다 먼저 나오는) `Utensil` 참조를 만나면 `Utensil.java` 파일을 살펴 `Utensil`과 `Dessert`를 모두 찾아냄 
>> 3) 두번째 명령 인수로 넘어온 `Dessert.java`를 처리하려고 할 때 같은 클래스 정의가 이미 있음을 알게 됨
>> 4) 컴파일 오류

```bash
javac Main.java 
javac Main.java Utensil.java
```
> 결과: pancake

```bash
javac Main.java Dessert.java
```
> 결과: potpie

어떤 소스를 먼저 컴파일하느냐에 따라서 동작이 달라지므로 반드시 수정이 필요하다 



### 해결방안 
서로 다른 소스 파일로 분리한다. 
```java
public class Test {
    public static void main(String[] args){
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    private static class Utensil {
        static final String NAME = "pan";
    }
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

```
핵심정리 
소스파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자
```
