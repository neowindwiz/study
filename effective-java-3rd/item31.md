# Item 31: Use bounded wildcards to increase API flexibility

## `List<String>`은 `List<Object>`의 하위 타입이 아니다.
* Item 28에 언급했듯이, 매개 변수화된 타입은 불변이다.
* Type1과 Type2은 `List<Type1>`의 하위 타입도 아니고, `List<Type2>`의 슈퍼 타입도 아니다.
* `List<Object>`에 어떤 객체도 넣을 수 있지만, `List<String>`에는 String만 넣을 수 있다.
* `List<String>`은 `List<Object>`가 할 수 있는 모듯 것을 할수 없기 때문에, 하위 타입이 아니다. (by the Liskov substitution principal, Item 10)

### Stack Class in Item 29
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}
```

### 일련의 원소들을 인자로 받아 차례로 스택에 집어넣는 메서드 정의

```java
// pushAll method without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}

Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);

StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
        numberStack.pushAll(integers);
                            ^
```
* Integer 형의 intVal로 push(intVal)을 호출하면 제대로 동작
* Integer는 Number의 하위 자료형(subtype)이기 때문이다.
* 논리적으로 보자면 아래의 코드는 문제가 없어야 할 것이지만, 실제로 해 보면 에러가 발생
* 앞서 설명한 대로, 형인자 자료형은 불변(invariant)이기 때문

### 문제해결 : 와일드카드 타입 적용

```java
// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```
* 자바는 이런 상황을 해결하기 위해, 한정적 와일드카드 자료형(bounded wildcard type)이라는 특별한 형인자 자료형을 제공
* pushAll의 인자 자료형을 "E의 Iterable"이 아니라 "E의 하위 자로형의 Iterable"이라고 명시할 방법이 필요한데, 와일드카드 자료형을 써서 Iterable<? extends E>라고 하면 된다는 것

### 와일드카드 자료형 없이 구현한 popAll 메서드

```java
// popAll method without wildcard type - deficient!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}

Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
```

* popAll 메서드는 스택의 모든 원소를 꺼내서 인자로 주어진 컬렉션에 넣는다.
* 이 메서드는 깔끔하게 컴파일될 뿐 아니라 인자로 주어진 컬렉션의 원소 자료형이 스택의 원소 자료형과 일치할 때는 완벽히 동작한다.
* 하지만 스택에서 원소를 꺼내는 코드는 에러가 발생한다.
* `Collection<Object>`가 `Collection<Number>`의 하위 자료형이 아니라는 오류가 난다.

### 문제해결 : 와일드카드 타입 적용
```java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```
* popAll의 인자 자료형을 "E의 컬렉션"이 아니라 "E의 상위 자료형(supertype)의 컬렉션"이라고 명시하면 된다.

### 여기 까지의 교훈
* 유연성을 최대화하려면, 객체 생산자(producer)나 소비자(consumer) 구실을 하는 메서드 인자의 자료형은 와일드카드 자료형으로 하라는 것이다.
* 어떤 와일드카드를 쓸지 암기하기 어렵다면, PECS (Produce - Extends, Consumer - Super) 약어를 활용하자.

### 이 니모닉을 염두에 두고 이 장의 이전 항목에서 제시한 몇 가지 방법과 생성자 선언을 살펴보자. 항목 28의 선택자 생성자는 다음과 같은 선언을합니다.

```java
public Chooser(Collection<T> choices)
```

```java
// Wildcard type for parameter that serves as an T producer
public Chooser(Collection<? extends T> choices)
```

## 명시적 형인자

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

```java
public static <E> Set<E> union(Set<? extends E> s1,
                               Set<? extends E> s2)
```

```java
Set<Integer>  integers =  Set.of(1, 3, 5);
Set<Double>   doubles  =  Set.of(2.0, 4.0, 6.0);
Set<Number>   numbers  =  union(integers, doubles);
```

```java
Union.java:14: error: incompatible types
        Set<Number> numbers = union(integers, doubles);
                                   ^
  required: Set<Number>
  found:    Set<INT#1>
  where INT#1,INT#2 are intersection types:
    INT#1 extends Number,Comparable<? extends INT#2>
    INT#2 extends Number,Comparable<?>
```

```java
// Explicit type parameter - required prior to Java 8
Set<Number> numbers = Union.<Number>union(integers, doubles);
```
* 컴파일러가 자료형을 명확하게 유추하지 못할 경우 사용

```java
public static <T extends Comparable<T>> T max(List<T> list)
```

```java
public static <T extends Comparable<? super T>> T max(
        List<? extends T> list)
```

```java
List<ScheduledFuture<?>> scheduledFutures = ... ;
```

```java
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

```java
public static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}

Swap.java:5: error: incompatible types: Object cannot be
converted to CAP#1
        list.set(i, list.set(j, list.get(i)));
                                        ^
  where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
```

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

# 요약
* API에서 와일드카드 타입을 사용하는 것은 까다롭지만, API를 더 유연하게 만든다.
* 범용 라이브러리를 만든다면, 와일드카드 타입을 적절하게 사용하는 것이 필수이다.
* 기본 규칙을 기억해라 : producer-extends, consumer-super (PECS)
* 또한 모든 `comparables`와 `comparators`은 `consumer`라는 점을 기억해라.