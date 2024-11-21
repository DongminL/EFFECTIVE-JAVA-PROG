# Item 13. clone 재정의는 주의해서 진행하라

## Cloneable 인터페이스란?

`Cloneable`은 **복제 가능한 클래스임을 명시**하는 용도의 믹스인 인터페이스다. 
명시 용도이기에 **메서드가 없는** 마커 인터페이스이기도 하다.

> 💡 믹스인 인터페이스(Mixin Interface) : 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 주는 인터페이스다. <br>
ex) `Comparable`은 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스다.

<br>

### Cloneable 사용 이유
---

1. `Object` 클래스의 `protected` 메서드인 `clone`을 `public` 메서드로 **개방**하기 위함이다. (외부 객체에서 사용할 수 없기 때문)
2. `Object`의 `clone` 메서드는 `Cloneable`을 구현하지 않고 `clone` 메서드를 호출하면 `CloneNotSupportedException` 예외를 던진다.
3. `super.clone()` 메서드를 사용해야 할 때 필요하다. (사용하지 않으면 필요 없음)
4. 사용자는 복제가 제대로 이뤄질 것이라고 기대할 수 있다.

따라서 `Cloneable`은 `Object` 클래스에 정의된 `clone` 메서드의 **동작 방식을 변경**하기 위함이다.

<br>

## 허술한 clone 메서드의 일반 규약 in Object 명세

**허술한 `Object`의 `clone` 메서드의 설계**로 인해, `Cloneable` 또한 주의해서 사용해야 한다.

다음은 `Object` 명세의 주요 허점이다.

> 이 객체의 복사본을 생성해 반환한다. '복사'의 정확한 뜻은 그 객체를 구현한 클래스에 따라 다를 수 있다. <br><br>
어떤 객체 `x`에 대해 <br>
`x.clone() != x`, `x.clone().getClass() == x.getClass()`, `x.clone().equals(x)` 식은 참이다. <br>
하지만 위 요구를 반드시 만족해야 하는 것은 아니다.

`clone` 메서드가 `super.clone()`이 아닌, 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러 상으로는 문제 없다. 또한 `super.clone()`을 연쇄적으로 호출하도록 구현하면 상위 클래스의 객체가 만들어져 결국 하위 클래스의 `clone` 메서드가 제대로 동작하지 않을 수 있다.

즉, **강제성이 없고** **생성자 연쇄(constructor chaining)**와 비슷한 메커니즘으로 구현될 때 문제가 발생할 수 있다.

<br>

## 불변 클래스의 경우

`final class`라면 걱정해야 할 하위 클래스가 없으니 `Cloneable`을 구현할 이유는 없다. 복제가 필요하면 **기존 객체를 재사용**하면 되기 때문이다.

그러나 `Cloneable`을 구현하고 싶다고 해보자.

``` java
public final class PhoneNumber implements Cloneable {
    private final short areaCode, prefix, lineNum;

    ...

    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();  // 발생하지 않음! (Cloneable을 구현하였기 때문)
        }
    }
}
```

1. `super.clone()`을 통해 복제를 한다.
2. `Object`의 `clone` 메서드는 `Object` 객체를 반환하지만, 공변 반환 타이핑(covariant return typing)을 통해 사용자가 형변환하지 않아도 되게끔 `PhoneNumber` 객체를 반환하게 한다. 이 방식을 권장한다.
3. `try-catch`로 감싼 이유는 `Object`의 `clone` 메서드가 `CloneNotSupportedException`를 던지기 때문이다.
4. 복제한 `PhoneNumber`를 반환하면 된다.

다시 한 번 강조하지만 **불변 클래스**는 굳이 **`clone` 메서드를 제공하지 않는 것**이 좋다.

<br>

## 클래스가 가변 객체를 참조하는 경우

가변 객체를 참조하는 경우, `clone` 메서드를 단순히 `super.clone()`의 결과를 그대로 반환하게 되면 주소값만 복사하게 되어 문제가 생길 수 있다. 결국 **얕은 복사**가 되어 **원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어 불변성을 해치게 된다.**

따라서 **복제**는 **원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변성을 보장해야 한다.**

``` java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    ...

    @Override
    public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

`Stack`의 내부까지 전부 복사를 하기 위한 가장 쉬운 방법은 `elements` 배열의 `clone` 메서드를 재귀적으로 호출하는 것이다.
위 코드에서 `elements.clone()`의 경우 형변환을 해주지 않았다. 그 이유는 배열의 `clone` 메서드는 런타임 타입과 컴파일타임 타입 모두 **원본 배열과 똑같은 배열을 반환**하기 때문이다. 

따라서 **배열은 복제(Clone) 기능을 제대로 사용**하는 유일한 예이기에 **배열의 `clone` 메서드를 사용하라고 권장**한다.

만약 `elements`가 `final` 필드라면, 새로운 값을 할당할 수 없어 앞선 방식은 작동하지 않는다. **복제를 해야 한다면 `final`을 제거해야 할 수도 있다.**

<br>

## 충분한 clone이 이루어지지 않을 경우, 깊은 복사를 하자

![Hashtable clone Method](https://velog.velcdn.com/images/milkskfk5677/post/3335d06d-80dc-4789-b114-3d4328fc8db7/image.png)

`Hashtable` 클래스의 `clone` 메서드는 얕은 복사로 지원한다. 이로 인해 원본과 같은 데이터를 참조하여 원본과 복제본 모두 변경되는 예기치 않은 문제가 발생할 가능성이 높다.

이를 해결하기 위해서는 **깊은 복사**를 지원하도록 보강해야 한다.

``` java
// 재귀적으로 호출하는 방식
private Entry deepCopy() {
    return new Entry(key, value,
        (next != null) ? next.deepCopy() : null);
}

// 반복문을 사용하는 방식
private Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

깊은 복사를 하는 `deppClone` 메서드를 구현하는 방법은 크게 2가지 있다. **재귀적** 방법, **반복문**을 이용하는 방법이 있다.

재귀적으로 호출하는 방법은 간단하지만, 데이터를 복제하는 과정에서 데이터의 깊이가 너무 깊다면 `Stack Overflow`를 일으킬 수 있다. 이를 해결하기 위해서는 반복문을 써서 순회하는 것으로 수정해야 한다.

``` java
public class HashTable<K, V> extends Hashtable<K, V> {

    // 깊은 복사 수행 (고수준 API 이용)
    public synchronized HashTable<K, V> deepClone() {
        HashTable<K, V> cloned = new HashTable<>();
        for (Map.Entry<K, V> entry : this.entrySet()) {
            cloned.put(deepCloneObject(entry.getKey()), deepCloneObject(entry.getValue()));
        }
        return cloned;
    }

    // 키와 값도 깊은 복사하기
    private <T> T deepCloneObject(T obj) {
        if (obj instanceof Cloneable) {
            try {
                // 리플렉션을 사용해 Cloneable을 구현한 객체의 clone() 메서드 호출
                return (T) obj.getClass().getMethod("clone").invoke(obj);
            } catch (Exception e) {
                throw new AssertionError("Clone not supported for: " + obj);
            }
        }
        // Cloneable이 아닌 경우 원본 반환
        return obj;
    }
}
```

이 외에도 `Collection`이나 `Map` 등 **고수준 API**를 활용하는 방법도 있다. 

이는 **간단하고 우아한 코드**로 작성할 수 있지만, 저수준에서 바로 처리할 때보다는 **느리다**. 또한 클래스 내부에서 각 필드를 세부적으로 복사하지 않고도, **데이터 구조 자체를 고수준으로 복제**하기 때문에, **`Cloneable` 아키텍처와는 어울리지 않는 방식**이다.

<br>

## clone 시, 변환 생성자나 변환 팩터리를 이용하자

`Cloneable`을 구현하면 이처럼 복잡하다. 물론 이처럼 복잡한 경우는 드물지만, `Cloneable`을 구현한 클래스를 확장해야 한다면 어쩔 수 없이 `clone` 메서드가 잘 작동되도록 구현해야 한다.

`Cloneable`을 구현하지 않는 경우라면, **변환 생성자**나 **변환 팩터리**를 이용하자!

``` java
// HashMap 클래스의 변환 생성자
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

// 변환 생성자를 모방한 HashMap 클래스의 변환 팩터리
public static <K, V> HashMap<K, V> fromMap(Map<? extends K, ? extends V> source) {
    HashMap<K, V> newMap = new HashMap<>(); // 새로운 빈 HashMap 생성
    newMap.putMapEntries(source, false); // 원본 Map의 엔트리 복사
    return newMap;
}
```

### Cloneable보다 좋은 점
---

- 위험한 객체 생성 메커니즘(생성자를 사용하지 않는 방법)을 사용하지 않는다.
- 허술한 규약을 따르지 않는다.
- 정상적인 `final` 필드 용법과도 충돌하지 않는다.
- 불필요한 예외를 던지지 않는다.
- 형변환도 필요하지 않다.
- 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.  

원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

<br>

## 마무리

> 새로운 인터페이스를 만들 때는 절대 `Cloneable`을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다!! <br>
'**복제 기능은 생성자와 팩터리를 이용**하는 게 최고'라는 것이다! 단, 예외적으로 **배열만은 `clone` 메서드 방식이 가장 깔끔한 방식**이다.