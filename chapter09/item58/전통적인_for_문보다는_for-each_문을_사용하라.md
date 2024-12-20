# Item 58. 전통적인 for 문보다는 for-each 문을 사용하라

## 전통적인 for 문의 단점

``` java
// 컬렉션 순회
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.netx();
    ... // e로 무언가를 한다.
}

// 배열 순회
for (int i = 0; i < a.length; i++) {
    ... // a[i]로 무언가를 한다.
}
```

- **반복자**와 **인덱스 변수**는 모두, **코드를 지저분하게** 할 뿐 우리에게 진짜 필요한 건 원소들 뿐이다.

- 혹시라도 잘못된 변수를 사용했을 때 **컴파일러가 오류를 잡아줄 수 없을지도** 모른다.

<br>

## 향상된 for 문 (for-each)

``` java
for (Element e : elements) {
    ... // e로 무언가를 한다.
}
```

**전통적인 for 문의 문제는 for-each 문을 사용하면 모두 해결**된다.

### for-each 문의 특징
---

- 반복자와 인덱스 변수를 사용하지 않아 **코드가 깔끔**해지고 **오류가 날 일도 없다**.

- **컬렉션과 배열을 모두 처리**할 수 있어, **어떤 컨테이너를 다루는지 고려하지 않아**도 된다.

- for-each 문을 사용해도 **성능 저하는 발생하지 않는다**.

### for-each 문으로 리팩토링
---

#### 전통적인 for 문 사용 시, 버그 발생 예시

``` java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
    NINE, TEN, JACK, QUEEN, KING }

...

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

...

List<Card> deck = new ArrayList<>();

// 전통적인 for문으로 버그 발생
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next())); // i.next()로 인해 suits가 더이상 없으면 NoSuchElementException 발생
```

#### 해결책 - 전통적인 for 문

``` java
// 위 코드는 동일

// 코드 58-6 문제는 고쳤지만 보기 좋진 않다. 
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    Suit suit = i.next();   // 바깥 반복문의 원소를 저장하는 변수 추가

    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(suit, j.next())); // i.next()로 인해 suits가 더이상 없으면 NoSuchElementException 발생
}
```

#### 해결책 - for-each 문

``` java
// 위 코드는 동일

// 코드 58-7 컬렉션이나 배열의 중첩 반복을 위한 권장 관용구 (349쪽)
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

**for-each 문을 중첩**하는 것으로 위 문제는 **간단히 해결**된다.

<br>

## for-each 문을 사용할 수 없는 상황

### 파괴적인 필터링
---

**컬렉션을 순회하면서 선택된 원소를 제거**해야 한다면 반복자의 **`remove` 메서드를 호출**해야 한다. **자바 8부터는 `Colloection`의 `removeIf` 메서드를 사용해** 컬렉션을 **명시적으로 순회하는 일을 피할 수 있다**.

### 변형
---

리스트나 배열을 순회하면서 그 **원소의 값 일부 혹은 전체를 교체**해야 한다면 리스트의 반복자나 배열의 **인덱스를 사용**해야 한다.

### 병렬 반복
---

**여러 컬렉션을 병렬로 순회**해야 한다면 각각의 **반복자와 인덱스 변수를 사용**해 **엄격하고 명시적으로 제어**해야 한다.

<br>

## 마무리

- 전통적인 for 문과 비교했을 때 **for-each 문**은 **명료하고, 유연하고, 버그를 예방**해준다. 심지어 **성능 저하도 없다**.

- **가능한 모든 곳에서 for 문이 아닌 for-each 문을 사용하자**.

- 원소들의 묶음을 표현하는 타입을 작성해야 한다면,  **`Iterable`을 구현하여 for-each 문을 사용할 수 있도록** 하자.
