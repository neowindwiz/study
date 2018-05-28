# Item 24: Favor static member classes over nonstatic

## 중첩(nested) 클래스는 다른 클래스 안에 정의된 클래스, 4가지 종류가 있다.
* static member classes
* nonstatic member classes
* anonymous classes
* local classes
* ```첫 번째를 제외한 나머지는 내부(inner) 클래스이다.```

## static member classes
* 가장 간단한 중첩 클래스이다.
* 외부 클래스의 객체 참조를 통해 모든 멤버를 직접 접근할 수 있다.
* 외부(outer) 클래와 함께 helper class로 주로 사용된다.


```java
// Typical use of a nonstatic member class
public class MySet<E> extends AbstracSet<E> {
    ... // Bulk of the class omitted

    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}