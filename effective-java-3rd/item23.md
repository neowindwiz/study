# Item 23: Prefer class hierachies to tagged classes

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

* 이런 태그 클래스들은 많은 단점을 갖고 있다.

# 용어
* `boilerplate code`
<pre><code>adfadsfasd</code></pre>

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

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

# 요약
* 태그 클래스는 드물게 적절할 수 있다.
* 명시적 태그 필드가 있는 클래스를 작성하려는 경우, 태그를 제거하고 클래스를 계층 구조로 대체할 수 있는지 여부를 고려해라.