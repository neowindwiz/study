# Item 23: Prefer class hierachies to tagged classes

## 다수의 기능을 갖고, 그 중 어떤 기능을 제공하는지 표시하는 태그(?)가 있는 클래스

```java
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
    final Shape shape;

    // Thes fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.lenght = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

### 이런 태그가 지정된 클래스들은 많은 단점을 갖고 있다.
* boilerplate code 반복
* 서로 다른 기능들이 한 클래스에 모여, 가독성이 떨어짐
* 필요없는 필드와 메서드가 생성되어 메모리 요구량 증가
* 생성자에 필요한 필드를 정확하게 초기화 해야됨, 런타임 오류 발생
* 새로운 기능 추가 시, swtich ~ case에도 추가해야됨
* 객체의 자료형으로 기능을 유추할 수 없음

### 결론적으로 태그가 지정된 클래스는 `장황하고`, `오류가 발생하기 쉽고`, `비효율적`이다.
- - -
## 태그가 지정된 클래스를 클래스 계층 구조로 만듬

```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override double area() { return length * width; }
}
```

* 태그로 달라지는 메서드를 추상 클래스의 추상 메서드로 만듬
* 공통으로 사용하는 필드 추가
* 태그가 지정된 클래스 기능을 하위 클래가 구현
* 난잡한 코드가 없고, 오류 발생률도 적다.
* 자료형으로 기능을 유추 가능하다. 
* 클래스가 계층구조를 갖고, 가독성도 높아진다.

## 정사각형(Square)는 사각형(Rectangle)의 한 종류임을 알 수 있다.

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

# 요약
* 태그 클래스는 드물게 적절할(?) 수 있다.
* 명시적 태그 필드가 있는 클래스를 작성하려는 경우, 태그를 제거하고 클래스를 계층 구조로 대체할 수 있는지 여부를 고려해라.