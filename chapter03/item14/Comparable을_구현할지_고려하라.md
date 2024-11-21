# Comparable을 구현할지 고려하라

## Comparable 인터페이스란?

``` java
public interface Comparable<T> {
    int compareTo(T o);
}
```

`Comparable`는 `compareTo`를 단일 메서드로 가지는 인터페이스다. 주로 **객체의 순서를 지정**할 때 사용된다. `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 **자연적인 순서가 있음**을 의미한다.

`Arrays.sort(a)`처럼 객체의 배열을 손쉽게 정렬할 수 있다.

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거 타입이 `Comparable`을 구현했다.

``` java
public class WordList {
    public static void main(String[] args) {
        // (중복을 제거하고) 알파벳순을 출력
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

`String`이 `Comparable`을 구현한 덕분엔 `TreeSet`으로 정렬이 가능하다.

순서가 명확한 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스를 구현하자.

<br>

## compareTo 메서드의 일반 규약

`compareTo` 메서드의 일반 규약은 `equals`의 규약과 비슷하다.

타입이 다른 객체에 대해서는 특별히 처리할 필요가 없다. **비교할 수 없는 타입**의 객체가 주어지면 간단히 `ClassCastException`을 던지면 된다. <br>
또한 **공통 인터페이스를 기반**로 비교할 경우, **다른 타입의 객체 간 비교도 허용**된다.

### 비교 결과에 따른 규약

- **x < y** : `x.compareTo(y)` < 0 <br>
- **x = y** : `x.compareTo(y)` == 0 <br>
- **x > y** : `x.compareTo(y)` > 0

### 주요 특징

- **반사성 (Reflexivity)** : <br>
모든 객체 `x`에 대해, `x.compareTo(x) == 0`이 성립해야 한다.

- **대칭성 (Symmetry)** : <br>
두 객체 `x`, `y`에 대해, `x.compareTo(y) == -y.compareTo(x)`가 성립해야 한다.

- **추이성 (Transitivity)** : <br>
객체 `x`, `y`, `z`가 있을 때, `x.compareTo(y) > 0`이고 `y.compareTo(z) > 0`이면, `x.compareTo(z) > 0`이 성립해야 한다.  

- **일관성 (Consistency)** : <br>
`x.compareTo(y) == 0`이면, `x.equals(y) == true`여야 한다.<br>
필수는 아니지만 권장한다!

### 왜 일관성을 권장할까?
---

일관성은 `compareTo`로 만든 순서와 `equals`의 결과가 일관되게 만든다.

두 결과가 일관되지 않아도 동작은 하지만, 이 클래스의 객체를 **정렬된 컬렉션에 넣으면 해당 컬렉이 구현한 인터페이스(`Collection`, `Set`, `Map` 등)에 정의된 동작이 엇박자** 낼 것이다. 

이를 통해 해당 인터페이스들은 `equals` 메서드의 규약을 따를 것 같지만, **정렬된 컬렉션들은 동치성을 비교할 때 `compareTo`를 사용**한다. 큰 문제는 아니지만, **주의**하도록 하자! 

다음은 `compareTo` 메서드가 일관성을 지키지 않았을 때의 결과이다.

``` java
@DisplayName("compareTo 일관성이 없을 때 문제")
@Test
void compareToTest() {
    BigDecimal value1 = new BigDecimal("1.0");
    BigDecimal value2 = new BigDecimal("1.00");
        
    // given
    HashSet<BigDecimal> hashSet = new HashSet<>();
    hashSet.add(value1);
    hashSet.add(value2);

    // when
    TreeSet<BigDecimal> treeSet = new TreeSet<>();
    treeSet.add(value1);
    treeSet.add(value2);

    // then
    assertAll(
            () -> assertNotEquals(value1.equals(value2), value1.compareTo(value2) == 0),
            () -> assertNotEquals(hashSet.size(), treeSet.size())
    );
}
```

`BigDecimal` 클래스는 `compareTo`의 일관성이 없다는 것을 확인할 수 있었다. 이로 인해, `HashSet`에서는 두 값 모두 추가가 됐는데, `TreeSet`에서는 `compareTo` 메서드를 통해 같은 값으로 인식하여 하나의 값만 추가되어 있었다.

이런 문제로 인해 일관성을 지키는 것을 권장한다.

### `compareTo` 메서드 작성 요령
---



### 새로운 컴포넌트를 추가할 때
---

구체 클래스에서 새로운 값 컴포넌트를 추가했다면 `compareTo` 규약을 지킬 수 없다. 그러나 **객체 지향적 추상화의 이점을 포기**한다면 **우회**할 수 있다.

#### 우회 방법

1. 확장 대신 독립된 클래스를 만든다.
2. 독립된 클래스에 원래 클래스의 인스턴스를 참조하는 필드를 둔다.
3. 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된다.

이렇게 하면 외부 클래스에 우리가 원하는 `compareTo` 메서드를 구현해넣을 수 있다. 필요에 따라 외부 클래스의 인스턴스를 필드 안에 담긴 원래 클래스의 인스턴스로 다룰 수도 있다.

<br>

## 기본 타입을 비교할 때, 비교자를 대신 사용하자

`compareTo` 메서드는 각 필드가 동치인지 비교하기 보다, 그 순서를 비교한다. 객체 참조 필드를 비교하려면 `compareTo` 메서드를 재귀적으로 호출한다. 

**`Comparable`을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(`Comparator`)를 대신 사용하자.**

``` java
public final class CaseInsensitiveString
        implements Comparable<CaseInsensitiveString> {

    private final String s;

    // 자바가 제공하는 Comparator를 사용해 클래스를 비교한다.
    public int compareTo(CaseInsensitiveString cis) {
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    }
    ...
}
```

### Wrapper 클래스의 `compare` 정적 메서드를 이용하자
---

`compareTo` 메서드에서 기본 타입 필드를 비교할 때, 관계 연산자 `>`, `<`를 사용하는 방식은 **가독성이 떨어진다**. 또한 **`float`이나 `double`의 경우, 비교가 정확하지 않을 수 있다.** 

그래서 박싱된 `Wrapper` 클래스들의 `compare` 정적 메서드를 이용하는 것이 좋다.

> ### Comparable vs Comparator: 언제 사용해야 할까?
>
> - `Comparable` : 특정 클래스의 기본적인 정렬 기준(자연스러운 순서)을 정의할 때 사용.
> - `Comparator` : 다양한 정렬 기준을 유연하게 적용하고 싶을 때 사용. (주로 익명 객체로 구현함)

<br>

## 정렬 기준이 여러 개일 때

- 핵심 필드가 여러 개라면 **가장 핵심적인 필드부터 비교**해나가자. 
- 비교 결과가 0이 아니라면, 그 결과를 바로 반환하자.
- 가장 핵심이 되는 필드가 똑같다면, 그 다음으로 중요한 필드를 비교해나간다.

``` java
// 코드 14-2 기본 타입 필드가 여럿일 때의 비교자 (91쪽)
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0)  {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

자바 8에서부터 **`Comparator` 인터페이스의 비교자 생성 메서드**를 이용해서 **메서드 체이닝 방식**으로 비교자를 생성할 수 있게 되었다. 이 메서드는 메서드 체이닝 방식과 더불어 **람다를 인수로** 받아 **코드가 더 깔끔**해진다.

``` java
// 코드 14-3 비교자 생성 메서드를 활용한 비교자 (92쪽)
private static final Comparator<PhoneNumber> COMPARATOR =
            // int 타입 키를 기준으로 순서를 정하는 비교자를 반환
            Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);
    
public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

이 방식은 간결하지만, **약간의 성능 저하**가 발생한다. 성능 저하가 발생하는 이유는 **람다식**과 **메서드 참조로 인한 오버헤드** 때문이다. 내 컴퓨터에서는 약 5% 정도의 성능 저하가 나타났다.

<br>

## '값의 차'를 기준으로 하는 비교자 - 오류 발생

``` java
static Comparator<Object> hashCodeOrder = 
    (Object o1, Object o2) -> o1.hashCode() - o2.hashCode();
```

**값의 차**를 기준으로 반환하게 되면 문제가 발생한다.

이 방식은 **정수 오버플로**가 발생하거나 **부동소수점 계산 방식에 따른 오류**를 낼 수 있다. 또한 위에서 설명한 방법대로 구현한 것에 비해 **월등히 빠르지도 않을 것이다**. 

**값의 차로 하지말고** 다음 두 방식 중 하나를 사용하자.

``` java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = 
    (Object o1, Object o2) -> Integer.compare(o1.hashCode(), o2.hashCode())
```

``` java
// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(Object::hashCode)
```

<br>

## 마무리

객체의 순서를 정하고 싶다면 꼭 `Comparable` 인터페이스를 구현하자. 이를 통해 쉽게 정렬과 검색을 하고, 비교 기능을 제공하는 컬렉션을 제대로 사용하자.

`compareTo` 메서드에서 필드의 값을 비교할 때 `<`, `>` 연산자는 사용하지 말자. <br>
대신 `Wrapper` 클래스가 제공하는 정적 `compare` 메서드나 `Comparator` 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.