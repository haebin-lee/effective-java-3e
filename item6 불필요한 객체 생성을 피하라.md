# item6. 불필요한 객체 생성을 피하라. 


이렇게 쓰지 말자
```java
String s  = new String("bikini");
```
이 문장은 실행될때 마다 String 인스턴스를 새로 만든다. 

이렇게 쓰자 
```java
String s = "bikini";
```
이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 나아가 이 방식을 사용한다면 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다. 


`Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드를 사용하는 것이 좋다. 생성자는 호출할 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않다. 

성능을 훨씬 더 끌어올릴 수 있다!
```java
static boolean isRomanNumeral(String s){
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3}" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
**String.matches 는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.**

Pattern은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine) 을 만들기 때문에 인스턴스 생성 비용이 높다. 

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3}" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
정규표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용한다. 

하지만 클래스가 초기화된 후 이 메서드를 한번도 호출하지 않는다면 ROMAN 필드는 쓸데 없이 초기화된 것이다. 그렇다고 지연초기화 코드를 추가하게 되면 코드를 복잡하게 만드는제 성능은 크게 개선되지 않기 때문에 추천하지 않는다. 

불필요한 객체를 만들어내는 또 다른 예로 `오토박싱(auto boxing)`을 들 수 있다.
**오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만 완전히 없애주는 것은 아니다** 

```java
private static long sum() {
    Long sum = 0L; 
    for(long i=0; i<= Integer.MAX_VALUE; i++){
        sum += i;
    }
    return sum; 
}
```
sum 변수를 `long`이 아닌 `Long`으로 선언해서 불필요한 Long 인스턴스가 약 2<sup>31</sup>개나 만들어 진 것이다. 
단순히 sum의 타입을 long으로만 바꿔주면 내 컴퓨터에서는 6.3초 에서 0.59초로 빨라진다. 
**박싱된 기본타입보다는 기본타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**

하지만 단순히 객체 생성을 피하고자 객체 풀(pool)을 만들지는 말자. 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 재사용하는 편이 낫다. 하지만 일반적으로는 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. 요즘 JVM의 가비지 컬렉터는 상당히 잘 최적화되어서 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.