# Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

## 컬렉션이 비었으면 null을 반환하지 말자

``` java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null  // null 반환
        : new ArrayList<>(cheesesInStock);
}
```

`null`을 반환한다면, 클라이언트는 이 `null` 상황을 처리하는 코드를 추가로 작성해야 한다. 아래와 같은 **방어 코드를 빼먹으면 오류가 발생**할 수 있다.

``` java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
    System.out.println("좋았어, 바로 그거야");
}
```

### null 값 반환을 옹호하는 입장에 대한 반박
---

- 컨테이너를 할당하는데도 비용이 드니 성능 저하가 될 것이라고 염려할 수 있지만, 주범이라고 확인되지 않는 것이라면 이 정도의 성능 차이는 신경 쓸 수준이 되지 않는다.

- 빈 컬렉션과 배열은 굳이 새로 할당하지 않고 반환할 수 있다.

<br>

## 빈 불변 컬렉션(또는 배열)을 사용하자

**`null` 값을 반환할 때의 문제점을 해결**하기 위한 효과적인 방법으로, **매번 똑같은 빈 '불변' 컬렉션(또는 배열)을 반환**하는 것이다.

``` java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()   // null 대신 빈 컬렉션 반환
        : new ArrayList<>(cheesesInStock);
}
```

``` java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

### 배열을 미리 할당하면 성능이 나빠진다
---

**단순히 성능을 개선할 목적이라면 `toArray`에 넘기는 배열을 미리 할당하는 건 추천하지 않는다**. 오히려 성능이 떨어진다는 연구 결과도 있다.

``` java
// 배열을 미리 할당하면 성능이 나빠진다
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

## 마무리

- **`null`이 아닌, 빈 배열이나 컬렉션을 반환하라!**

- **`null`을 반환하는 API는 사용하기 어렵**고 **오류 처리 코드도 늘어**난다. 그렇다고 성능이 좋은 것도 아니다.
