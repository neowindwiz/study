# Item 25: Limit source files to a single top-level class

* Java 컴파일러를 사용하면 단일 소스 파일에서 여러 최상위 클래스를 정의할 수 있지만, 이와 관련된 이점은 없으며 심각한 위험이 있다.
* 소스 파일에 여러 최상위 클래스를 정의하면 클래스에 대해 여러 정의를 제공할 수 있기 때문에 위험이 발생한다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

## 원본 파일이 컴파일러에 전달되는 순서에 따라 영향을 받는다.
```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```

```java
// Two classes defined in one file. Don't ever do this!
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

### 클래스의 정적 클래스로 변경으로 문제 해결

```java
// Static member classes instead of multiple top-level classes
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}