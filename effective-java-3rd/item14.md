# 아이템 14: Comparable 구현을 고려해라
Comparable 인터페이스를 구현한 클래스의 인스턴스는 `natural ordering`을 갖는다.
```java
Arrays.sort(a);
```

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

* sgn(x.compareTo(y)) == -sng(y.compareTo(x))
* x.compareTo(y) > 0 && y.compareTo(z) > 0
* x.compareTo(y) == 0, sgn(x.compareTo(z)) == sgn(y.compareTo(z))
* (x.compareTo(y) == 0) == (x.equals(y))

```java
// Single-field Compareable with object reference field
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ... // Remainder omitted
}
```

```java
// Multiple-field Comparable with primitive fields
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if(result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if(result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

```java
// Comparable with comparator construction methos
private static final Comparator<PhoneNumber> COMPARATOR = 
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

```java
// BROKEN difference-based comparator - violates transitivilty!
static Comparator<Object> hasCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

```java
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

```java
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```