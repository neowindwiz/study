# 아이템 14: Comparable 구현을 고려해라
* compareTo 함수는 객체에 선언되어 있지 않다.
* compareTo 함수는 Comparable 인터페이스의 유일한 함수 이다.
* Comparable 구현한 클래스의 인스턴스는 `natural 
ordering`을 갖는다.

### 간단한 배열 비교

```java
Arrays.sort(a);
```

### 중복 제거 문자열 출력
TreeSet은 sroted collection으로 중복을 허용하지 않는다.
```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

### Comparable 인터페이스
자바 플랫폼 라이브러리의 value class들과 모든 enum type [(Item 34)](item34.md)들은 Comparable 구현한다.
```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

### compareTo 는
* 일반적으로 equals 과 유사하다.
* 객체보다 작을 떄는 음수 반환, 같으면 0 반환, 크면 양수 반환.
* 객체의 타입이 다른 경우, `ClassCastException` 반환
* sgn(x.compareTo(y)) == -sng(y.compareTo(x))
* x.compareTo(y) > 0 && y.compareTo(z) > 0, x.compareTo(z) > 0
* x.compareTo(y) == 0, sgn(x.compareTo(z)) == sgn(y.compareTo(z))

* (x.compareTo(y) == 0) == (x.equals(y))

### Hashcode로 비교할 수 없을때
크거나 작고 같은 지를 비교하기 위해서는 compareTo를 사용
* TreeSet이나 TreeMap 같은 sorted collection
* Arrays와 Collections같은 utility class
* 탐색과 정렬 알고리즘을 포함하는 class


### equals 와 compareTo
hashSet은 equals로 비교, treeSet은 compareTo
```java
//hashSet
  Set<BigDecimal> hashSet = new HashSet<>();

  hashSet.add(new BigDecimal("1.0"));
  hashSet.add(new BigDecimal("1.00"));
  System.out.println(hashSet.size());  //2


  //treeSet
  Set<BigDecimal> treeSet = new TreeSet<>();

  treeSet.add(new BigDecimal("1.0"));
  treeSet.add(new BigDecimal("1.00"));
  System.out.println(treeSet.size());  //1
```
```java
System.out.println(new BigDecimal("1.0").compareTo(new BigDecimal("1.00")));
//0

System.out.println(new BigDecimal("1.0").equals(new BigDecimal("1.00")));
//false

System.out.println(new Double(1.0).equals(1.00));
//true
```

```java
// Single-field Compareable with object reference field
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ... // Remainder omitted
}
```

* primitive fields는 <, > 연산 보다는, Double.compre, Float.Compare를 사용하는걸 추천

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

### 성능 향상을 위해
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

### 사용하지 말자 (integer overflow)
* Integer.MAX_VALUE를 넘어버리면 음수가 된다.
* 에러는 발생하지 않음...
```java
// BROKEN difference-based comparator - violates transitivilty!
static Comparator<Object> hasCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

### 사용하자 (static compare method)
```java
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

### 사용하자 (construction method)
```java
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

### 정리
* Compareable 인터페이스는 정렬, 탐색
* <, > 연상 사용을 피한다.