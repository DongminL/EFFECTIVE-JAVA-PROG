# Item 89. 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

## 싱글톤 패턴 직렬화하기

``` java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis { ... }

    private void leaveTheBuilding() { ... }
}
```

이 클래스는 외부에서 생성자를 호출하지 못하게 막아 인스턴스가 오직 하나만 만들어짐을 보장하는 **싱글톤 패턴**이다.

그러나 이 클래스에 `implements Serializable`을 추가하면 더 이상 싱글톤 패턴이 아니게 된다. **어떤 `readObject`를 사용하더라도 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환**하게 된다.

**`readResolve` 기능을 이용하면 `readOject`가 만들어낸 인스턴스를 다른 것으로 대체**할 수 있다. 역직렬화한 객체의 클래스가 `readResolve` 메서드를 적절히 정의했다면, 역직렬화 후 새롭게 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다. 대부분의 경우 이때 새롭게 생성된 객체의 참조는 유지하지 않아 바로 가비지 컬렉션 대상이 된다.

<br>

## 잘못된 싱글톤 패턴 사용 시 문제점
---

``` java
// 잘못된 싱글톤 - transient가 아닌 참조 필드를 가지고 있다!
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis {  }

    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

**`readResolve`를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 `transient`로 선언해야 한다**. 그렇지 않으면 `MutablePeriod` 공격과 비슷한 방식으로 `readResolve` 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.

### 잘못된 싱글톤 패턴을 공격하는 방법
---

공격 방법은 복잡하지만 기본 아이디어는 간단하다. 싱글톤이 `transient`가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 `readResolve` 메서드가 실행되기 전에 역직렬화된다. 그렇다면 잘 조작된 스트림을 써서 해당 **참조 필드의 내용이 역직렬화되는 시점에 그 역렬화된 인스턴스의 참조를 훔쳐올 수 있다**.

``` java
// 도둑 클래스
public class ElvisStealer implements Serializable {

    private static final long serialVersionUID = 0;

    static Elvis impersonator;  // 훔친 인스턴스를 저장할 필드
    private Elvis payload;  // Elvis 인스턴스를 받을 필드

    private Object readResolve() {
		// resolve되기 전의 Elvis 인스턴스의 참조를 저장한다. (훔쳐서 보관)
        impersonator = payload;
		// favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
        return new String[]{"A Fool Such as I"};
    }
}
```

**이 인스턴스의 필드는 도둑이 '숨길' 직렬화된 싱글톤을 참조하는 역할**을 한다. 직렬화된 스트림에서 싱글톤의 비휘발성 필드를 이 도둑의 인스턴스로 교체한다. 이렇게 하면 싱글톤은 도둑을 참조하고 도둑은 싱글톤을 참조하는 **순환고리**가 만들어진다.

싱글톤이 도둑을 포함하므로 싱글톤이 역직렬화될 때 도둑의 `readResolve` 메서드가 먼저 호출된다. 이로 인해, 도둑의 `readResolve` 메서드가 수행될 때 도둑의 인스턴스 필드에는 역직렬화 도중인 싱글톤의 참조가 담겨 있게 된다.

도둑의 `readResolve` 메서드는 이 인스턴스 필드가 참조한 값을 정적 필드로 복사하여 `readResolve`가 끝난 후에도 참조할 수 있게 한다. 그런 다음 이 메서드는 도둑이 숨긴 `transient`가 아닌 필드의 원래 타입에 맞는 값을 반환한다. **이 과정을 생략하면 직렬화 시스템이 도둑의 참조를 이 필드에 저장하려고 할 때 VM이 `ClassCastException`을 던진다**.

### 수작업으로 만든 스트림 사용하는 방법
---

수작으로 만든 스트림을 이용해 2개의 싱글톤 인스턴스를 만들어낸다.

``` java
// 직렬화의 허점을 이용해 싱글턴 객체를 2개 생성한다.
public class ElvisImpersonator {
    // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림
    private static final byte[] serializedForm = {
            -84, -19, 0, 5, 115, 114, 0, 20, 107, 114, 46, 115,
            ... // 생략
            32, 72, 111, 116, 101, 108
    };

    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화한 다음,
        // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
        Elvis elvis = (Elvis) deserialize(serializedForm);
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites();  // 같은 Elvis 인스턴스를 두 개 가짐
    }

    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(
                new ByteArrayInputStream(sf)
            ).readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

이 프로그램을 실행하면 다음과 같이 서로 다른 2개의 `Elvis` 인스턴스를 생성하고 출력한다.
``` java
[Hound Dog, Heartbreak Hotel]   // 원래 Elvis.INSTANCE 
[A Fool Such as I]  // ElvisStealer에서 훔친 클래스
``` 

<br>

## 해결책 1 - transient 사용하기

`favoriteSongs` 필드를 `transient`로 선언하여 이 문제를 고칠 수 있지만 `Elvis`를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다. **`readResolve` 메서드를 사용해 '순간적으로' 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉽고 신경을 많이 써야 하는 작업**이다.

<br>

## 해결책 2 - 열거 타입 사용하기 (권장)

**직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장**해준다. 물론 임의의 네이티브 코드를 수행할 수 있는 특권을 가로챈 공격자에게는 모든 방어가 무력화된다.

``` java
// 코드 89-4 열거 타입 싱글턴(선호 방식) - 전통적인 싱글턴보다 우수하다. (478쪽)
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

인스턴스 통제를 위해 `readResolve` 메서드를 사용하는 방식이 완전히 쓸모 없는 것은 아니다. **직렬화 가능 인스턴스 통제 클래스를 작성해야 하는데, 컴파일타임에는 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능**하기 때문이다.

<br>

## fianl이 아닌 클래스에서 readResolve 접근성의 주의점

**`readResolve` 메서드에 접근성은 매우 중요**하다. `fainl` 클래스라면 `readResolve` 메서드는 `private`이어야 한다. `final`이 아닌 클래스에서는 몇 가지를 주의해서 고려해야 한다.

- **`private`으로 선언하면 하위 클래스에서 사용할 수 없다**.

- **`protected`나 `public`으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다**.

- **`protected`나 `public`이면서 하위 클래스에서 재정의하지 않았다면, 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 `ClassCastException`을 일으킬 수 있다**.

따라서 **`final`이 아닌 클래스라면 `readResolve` 메서드를 `private`으로 만들고, 하위 클래스에서 필요한 경우 별도로 정의하도록 해야 한다**. 이렇게 하면 하위 클래스에서 `readResolve`를 재정의를 하지 않아 발생하는 오류를 줄일 수 있다.

<br>

## 정리

- **불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자**.

- 여의치 않은 상황에서 **직렬화와 인스턴스 통제가 모두 필요하다면 `readResolve` 메서드를 작성해 넣어**야 하고, **그 클래스에서 모든 참조 타입 인스턴스 필드를 `transient`로 선언**해야 한다.
