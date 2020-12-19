# item16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.

이처럼 퇴보한 클래스는 public이어서는 안된다.
```java
class Point {
    public double x;
    public double y;
}
```

접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화 한다. 
```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }
    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```
패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로서 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

하지만, package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제는 없다. 

Public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 결코 좋은 생각이 아니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할수 없다는 단점은 여전하다. 

불변 필드를 노출한 public 클래스- 과연 좋은가?
```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour; 
    public final int minute;

    public Time(int hour, int minute){
        if( hour <0 || hour >= HOURS_PER_DAY )
            throw new IllegalArgumentException("시간: "+hour);
        if (minute <0 || minutes >= MINUTES_PER_HOUR) 
            throw new IllegalArgumentException("분: "+minute );
        this.hour = hour; 
        this.minute = minute;
    }
}
```

```
핵심정리 
public 클래스는 절대 가변 필드를 직접 노출해서는 안된다. 불변필드라면 노출해도 덜 위험하겠지만 완전히 안심할 수 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.
```