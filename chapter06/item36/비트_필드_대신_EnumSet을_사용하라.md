# Item 36. 비트 필드 대신 EnumSet을 사용하라

## 비트 필드의 문제점

열거한 값들이 주로 집합으로 사용될 경우, **각 상수에 서로 다른 2의 거듭제곱 값을 할당**한 **정수 열거 패턴**을 사용해왔다.

**비트별 `OR`를 사용해 여러 상수를 하나의 집합**으로 모을 수 있고, 이렇게 만든 집합을 **비트 필드**(Bit Field)라고 한다. <br>

``` java
// 코드 36-1 비트 필드 열거 상수 - 구닥다리 기법! (223쪽)
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) { ... }

    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    }
}
```

### 비트 필드의 장점

- **합집합과 교집합 같은 집합 연산을 효율적으로 수행**할 수 있다.

### 비트 필드의 단점

- **정수 열거 상수의 단점**(아이템 34)을 그대로 지닌다.

- **비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다**.

- 비트 필드 하나에 녹아 있는 **모든 원소를 순회하기 어렵다**.

- API를 작성할 때 **최대 몇 비트가 필요한지 미리 예측하여 적절한 타입을 선택**해야 한다. <br>
API를 수정하지 않고 비트 수를 더 늘릴 수 없기 때문이다.


## 해결책 - EnumSet을 사용하자

### EnumSet이란

**열거형 원소**들을 쉽고 빠르게 사용하기 위한 `Set` 인터페이스의 특수 구현체다. `EnumSet`은 집합 생성 등 다양한 기능의 정적 팩터리를 제공한다.

### EnumSet의 장점

- 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.

- `Set` 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용할 수 있다.

- **원소가 총 64개 이하**라면, `EnumSet` 전체를 `long` 변수 하나로 표현하여 **비트 필드에 비견되는 성능**을 보여준다. <br>
이는 `EnumSet`의 내부가 비트 벡터로 구현되었기 때문이다.

- 난해한 작업을 `EnumSet`이 다 처리하기 때문에, **비트를 직접 다룰 때 흔히 겪는 오류들로부터 해방**된다. <br> 
`removeAll`과 `retainAll` 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 연산을 사용하여 구현되었기 때문이다.

#### 비트 필드를 EnumSet을 사용하여 리팩토링

``` java
// 코드 36-2 EnumSet - 비트 필드를 대체하는 현대적 기법 (224쪽)
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 인터페이스로 받는 게 일반적으로 좋은 습관이다.
    // 다른 Set 구현체도 처리 가능하나,, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

비트 필드에 비해 **짧고 깔끔**하며, **타입과 비트 연산들로부터 안전**하다.

### EnumSet의 유일한 단점

아직까지 **불변 `EnumSet`을 만들 수 없다**. 불변으로 사용하고 싶으면, `Collections.unmodifiableSet`으로 `EnumSet`을 감싸 사용할 수 있다. 다만 명확성과 성능이 조금 떨어지게 된다.
