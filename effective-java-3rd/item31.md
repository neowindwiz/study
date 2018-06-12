# Item 31: Use bounded wildcards to increase API flexibility

## `List<String>`은 `List<Object>`의 하위 타입이 아니다.

- 매개 변수화된 타입은 불변이다. (Item 28)
- Type1 과 Type2 은 `List<Type1>`의 하위 타입도 아니고, `List<Type2>`의 슈퍼 타입도 아니다.
- `List<Object>`에 어떤 객체도 넣을 수 있지만, `List<String>`에는 String 만 넣을 수 있다.
- `List<String>`은 `List<Object>`가 할 수 있는 모듯 것을 할수 없기 때문에, 하위 타입이 아니다. (by the Liskov substitution principal, Item 10)

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

- Integer 형의 intVal 로 push(intVal)을 호출하면 제대로 동작
- Integer 는 Number 의 하위 자료형(subtype)이기 때문이다.
- 논리적으로 보자면 아래의 코드는 문제가 없어야 할 것이지만, 실제로 해 보면 에러가 발생
- 앞서 설명한 대로, 형인자 자료형은 불변(invariant)이기 때문

### 문제해결 : 와일드카드 타입 적용

```java
// Wildcard type for a parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

- 자바는 이런 상황을 해결하기 위해, 한정적 와일드카드 자료형(bounded wildcard type)이라는 특별한 형인자 자료형을 제공
- pushAll 의 인자 자료형을 "E 의 Iterable"이 아니라 "E 의 하위 자로형의 Iterable"이라고 명시할 방법이 필요한데, 와일드카드 자료형을 써서 Iterable<? extends E>라고 하면 된다는 것

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

- popAll 메서드는 스택의 모든 원소를 꺼내서 인자로 주어진 컬렉션에 넣는다.
- 이 메서드는 깔끔하게 컴파일될 뿐 아니라 인자로 주어진 컬렉션의 원소 자료형이 스택의 원소 자료형과 일치할 때는 완벽히 동작한다.
- 하지만 스택에서 원소를 꺼내는 코드는 에러가 발생한다.
- `Collection<Object>`가 `Collection<Number>`의 하위 자료형이 아니라는 오류가 난다.

### 문제해결 : 와일드카드 타입 적용

```java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

- popAll 의 인자 자료형을 "E 의 컬렉션"이 아니라 "E 의 상위 자료형(supertype)의 컬렉션"이라고 명시하면 된다.

### 여기 까지의 교훈

- 유연성을 최대화하려면, 객체 생산자(producer)나 소비자(consumer) 구실을 하는 메서드 인자의 자료형은 와일드카드 자료형으로 하라는 것이다.
- 어떤 와일드카드를 쓸지 암기하기 어렵다면, PECS (Produce - Extends, Consumer - Super) 약어를 활용하자.
- Naftalin 과 Wadler 는 그것을 Get and Put Principle 이라고 부른다.

### 이 니모닉을 염두에 두고 이 장의 이전 항목에서 제시한 몇 가지 방법과 생성자 선언을 살펴보자. 항목 28 의 선택자 생성자는 다음과 같은 선언을합니다.

```java
[변경전]
public Chooser(Collection<T> choices)

[개선된]
// Wildcard type for parameter that serves as an T producer
public Chooser(Collection<? extends T> choices)
```

```java
[변경전]
public static <E> Set<E> union(Set<E> s1, Set<E> s2)

[개선된]
public static <E> Set<E> union(Set<? extends E> s1,
                               Set<? extends E> s2)
```

- s1 과 s2 의 두 매개변수는 모두 E 생성자이므로, PECS 니모닉으로 위와 같이 선언할 수 있다.

```java
Set<Integer>  integers =  Set.of(1, 3, 5);
Set<Double>   doubles  =  Set.of(2.0, 4.0, 6.0);
Set<Number>   numbers  =  union(integers, doubles);
```

- 적절한 용도로 사용되는 와일드카드 유형은 클래스 사용자에게 거의 보이지 않는다.
- 클래스 사용자가 와일드카드 유형에 대해 생각해야 한다면 API 에 문제가 있을 수 있습니다.
- (그들은 그들이 받아 들여야하는 매개 변수를 받아들이고 거부해야하는 매개 변수를 허용하는 방법을 만든다.)

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

- Java 8 이전에는 유형 추론 규칙이 이전 코드 조각을 처리 할만큼 영리하지 못했다.
- 컴파일러는 문맥 적으로 지정된 반환 유형 (또는 대상 유형)을 사용하여 E 유형을 추론 필요

```java
// Explicit type parameter - required prior to Java 8
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

- 컴파일러가 올바른 유형을 추론하지 못하면, 명시적 유형 인수 [JLS, 15.12]와 함께 사용할 유형을 항상 알 수 있도록 한다.
- 자바 8 에서 타깃 타이핑을 도입하기 전에도 이런 일은 자주 해야 하는 일이 아니다. (?)
- 명시적 유형 인수가 추가 된 경우 여기에 표시된 코드 조각은 Java 8 이전의 버전에서 깨끗하게 컴파일됨

```java
[이전]
public static <T extends Comparable<T>> T max(List<T> list)

[개선된]
public static <T extends Comparable<? super T>> T max(
        List<? extends T> list)
```

```java
List<ScheduledFuture<?>> scheduledFutures = ... ;

// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

- 이 두 선언 중 어느 것이 더 바람직한가? 공용 API 에서는 두 번째가 더 간단하기 때문에 더 좋다.
- 일반적으로 형식 매개 변수가 메소드 선언에 한 번만 나타나는 경우 해당 매개 변수를 와일드카드로 대체합니다.
- 제한 없는 유형 매개변수인 경우, 제한없는 와일드 카드로 대체됩니다.
- 경계 유형 매개 변수인 경우 경계 와일드 카드로 교체하십시오.
- 두 번째 스왑 선언에는 문제가 하나 있다. 직진 구현은 컴파일되지 않습니다.

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

- 우리가 방금 빼낸 요소들을 `List`에 다시 넣을 수 없다는 것은 옳지 않은 것처럼 보인다.
- 문제는 `List<?>`가 `List type` 이라는 것. null 을 제외한 모든 값을 목록에 넣을 수 없다.

### 안전하지 않은 캐스트 또는 원시 유형에 의존하지 않는 구현

```java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

- 와일드 카드 유형을 캡처하기 위한 Helper 작성
- `List<E>`는 이 List 에서 얻을 수 있는 값이 E 라는 것을 알 수 있다.
- 공개 방법에 비해 너무 복잡하다고 기각한 서명을 정확하게 가지고 있다는 것을 주목할 가치가 있다. (?)

# 요약

- API 에서 와일드카드 타입을 사용하는 것은 까다롭지만, API 를 더 유연하게 만든다.
- 범용 라이브러리를 만든다면, 와일드카드 타입을 적절하게 사용하는 것이 필수이다.
- 기본 규칙을 기억해라 : producer-extends, consumer-super (PECS)
- 또한 모든 `comparables`와 `comparators`은 `consumer`라는 점을 기억해라.
