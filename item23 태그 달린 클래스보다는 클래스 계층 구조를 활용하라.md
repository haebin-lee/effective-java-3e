# item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 문제점 
> 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다!
```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(Rectangle)일 때만 쓰인다.
    double length; 
    double width; 

    // 다음 필드들은 모양이 원(Circle) 일 때만 쓰인다. 
    double radius;

    // 원용 생성자 
    Figure(double radius){
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자 
    Figure(double length, double width){
        shape = Shape.RECTANGLE;
        this.length = length; 
        this.width = width;
    }

    double area() {
        switch(shape){
            case RECTANGLE :
                return length * width; 
            case CIRCLE: 
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
태그 달린 클래스의 단점 

1. 열거타입 선언, 태그 필드, switch 문 등 쓸데 없는 코드가 많다. 
2. 가독성도 나쁘다 
3. 다른 의미를 위한 코드도 함께 있어 메모리도 많이 사용한다. 
4. 엉뚱한 필드를 초기화해도 런타임때에 문제가 드러난다. 
5. 새로운 의미를 추가할 때마다 모든 switch 문을 수정해야 한다. 
6. **인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀없다.**

### 해결방법 
1. 가장 먼저 계층구조의 루트(root)가 될 추상 클래스를 정의한다. 
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.


> 태그 달린 클래스를 클래스 계층 구조로 변환 
```java
abstract class Figure {
    abstract double area(); 
} 
class Circle extends Figure {
    final double radius;
    Circle(double radius) {
        this.radius = radius;
    }
    @Override 
    double area() {
        return Math.PI * (radius * radius);
    }
}
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width){
        this.length = legnth; 
        this.width = width; 
    }
    @Override
    double area() {
        return legnth * width;
    }
}
```
장점 
1. 쓸데 없는 코드가 모두 없어졌다. 
2. 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했다. 
3. 각 클래서의 생성자가 모든 필드를 남김 없이 초기화 하고 추상메서드를 모두 구현했는지 컴파일러가 확인해준다. 
4. 실수로 빼먹은 case문 때문에 런타임 오류가 발생할 일도 없다. 


여기서 정사각형을 구현한다면 사각형의 특별한 형태임을 아주 간단하게 반영할 수 있다. 
```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

```
핵심 정리 
태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층 구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층 구조로 리팩터링 하는 걸 고민해보자.
```