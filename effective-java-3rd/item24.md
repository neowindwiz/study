# Item 24: Favor static member classes over nonstatic

# 4가지 종류의 중첩 클래스
* static member classes
* nonstatic member classes
* anonymous classes
* local classes

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