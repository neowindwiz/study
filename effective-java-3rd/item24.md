# Item 24: Favor static member classes over nonstatic

## 중첩(nested) 클래스는 다른 클래스 안에 정의된 클래스, 4가지 종류가 있다.
* static member classes
* nonstatic member classes
* anonymous classes
* local classes
* ```첫 번째를 제외한 나머지는 내부(inner) 클래스이다.```

## 정적 멤버 클래스

### 샘플코드 : https://docstore.mik.ua/orelly/java-ent/jnut/ch03_09.htm
```java
// A class that implements a stack as a linked list
public class LinkedStack {
    // This static member interface defines how objects are linked
    public static interface Linkable {
        public Linkable getNext();
        public void setNext(Linkable node);
    }

    // The head of the list is a Linkable object
    Linkable head;  

    // Method bodies omitted
    public void push(Linkable node) { ... } 
    public Object pop() { ... } 
}

// This class implements the static member interface
class LinkableInteger implements LinkedStack.Linkable {
    // Here's the node's data and constructor
    int i;
    public LinkableInteger(int i) { this.i = i; }

    // Here are the data and methods required to implement the interface
    LinkedStack.Linkable next;
    public LinkedStack.Linkable getNext() { return next; }
    public void setNext(LinkedStack.Linkable node) { next = node; }
}
```

* 가장 간단한 중첩 클래스이다.
* 외부 클래스의 객체 참조를 통해 모든 멤버를 직접 접근할 수 있다.
* 외부(outer) 클래와 함께 helper class로 주로 사용된다.
* enum을 정적 정적 멤버 클래스로, Calculator.Operation.PLUS 와 같이 사용할 수 있다.


## 비 정적 멤버 클래스

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
```

* 외부(Outer) 클래스 객체와 자동으로 연결된다.
* 외부 클래스의 메서드를 호출 할 수 있고, 참조(`this`)도 얻을 수 있다. 
* 외부 클래스 없이 존재 할 수 없다.
* 외부 클래스 객체 메서드에서 비 정적 멤버 클래스의 생성자를 호출할때, 생성된다.
* 외부 클래스 객체를 다르게 보이기 위해 어댑터를 제공할떄 자주 사용된다.
* `외부 클래스에 대한 참조를 유지해야 하며, 시간과 공간 요구량이 늘어난다.`
  * `외부 클래스의 GC가 힘들어진다.`
* `외부 클래스에 접근이 필요없다면, 정적클래스로 수정..`

## 익명 클래스

### 샘플코드 : https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html
```java
HelloWorld frenchGreeting = new HelloWorld() {
    String name = "tout le monde";
    public void greet() {
        greetSomeone("tout le monde");
    }
    public void greetSomeone(String someone) {
        name = someone;
        System.out.println("Salut " + name);
    }
};
```
* 외부 클래스의 멤버가 아니다.
* 사용할때, 선언하고 객체를 만들어 사용한다.
* 코드 어디서도 사용할 수 있다.
* lambda가 추가된 후, 익명 클래스 보다 lambda가 선호된다.

## 로컬 클래스

### 샘플코드 : https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html
```java
public void sayGoodbyeInEnglish() {
    class EnglishGoodbye {
        public static void sayGoodbye() {
            System.out.println("Bye bye");
        }
    }
    EnglishGoodbye.sayGoodbye();
}
```
* 잘 사용하지 않는다. (빈도가 적다.)
* 지역 변수와 같은 Scope를 갖는다.