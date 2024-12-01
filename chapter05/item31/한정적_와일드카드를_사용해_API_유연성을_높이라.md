매개변수화 타입은 불공변(invariant)  
-> `List<String>`은 `List<Object>`의 하위 타입이 아니라는 뜻

하지만 때론 불공변 방식보다 유연한 무언가가 필요하다.

## 한정적 와일드카드 타입
```java
public void pushAll(Iterable<E> src) {  
    for (E e : src) {  
        push(e);  
    }  
}
//
public static void main(String[] args) {  
	Stack<Number> numberStack = new Stack<>();  
	Iterable<Integer> integers = Arrays.asList(3, 1, 4, 1, 5, 9);  
	numberStack.pushAll(integers);  
}
```
`Integer`는 `Number`의 하위 타입이니 잘 동작해야 하지만 오류 메시지가 뜨게 됩니다.

```
error: incompatible types: Iterable<Integer> cannot be converted to Iterable<Number>
        numberStack.pushAll(integers);
                            ^
```

이런 상황에 대처할 수 있는 것이 한정정 와일드카드 타입이라는 특별한 매개변수화 타입입니다.

`pushAll`의 입력 매개변수 타입은 `E의 Iterable`이 아니라 `E의 하위 타입의 Iterable`이어야 함.

```java
public void pushAll(Iterable<? extends E> src) {  
    for (E e : src) {  
        push(e);  
    }  
}
```
수정을 마치면 Stack과 클라이언트 코드에서 깔끔하게 컴파일이 됩니다.  
컴파일이 되었다는건 모든 것이 타입 안전하다는 뜻.

```java
public void popAll(Collection<E> dst) {  
    while (!isEmpty())  
        dst.add(pop());  
}

///

Collection<Object> objects = new ArrayList<>();  
numberStack.popAll(objects);
```

```
error: incompatible types: Collection<Object> cannot be converted to Collection<Number>
        numberStack.popAll(objects);
                           ^
```
`popAll` 코드를 컴파일하면 에러가 발생.

이번에는 입력 매개변수의 타입이 `E의 Collection`이 아니라 `E의 상위 타입의 Collection`이어야 함.

```java
public void popAll(Collection<? super E> dst) {  
    while (!isEmpty())  
        dst.add(pop());  
}
```
수정을 마치면 깔끔하게 컴파일됩니다.

**유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.**  
만약 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 쓰는게 아니라 타입을 정확히 지정해야 하는 상황.

>PECS: producer-extends, consumer-super

매개변수화 타입T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용하라는 뜻.

### PECS 사용 예시

```java
// 코드 31-5 T 생산자 매개변수에 와일드카드 타입 적용 (184쪽)  
public Chooser(Collection<? extends T> choices) {  
    choiceList = new ArrayList<>(choices);  
}

public static void main(String[] args) {  
    List<Integer> intList = List.of(1, 2, 3, 4, 5, 6);  
    Chooser<Number> chooser = new Chooser<>(intList);  
    for (int i = 0; i < 10; i++) {  
        Number choice = chooser.choose();  
        System.out.println(choice);  
    }  
}
```

```java
public static <E> Set<E> union(Set<? extends E> s1,  
							   Set<? extends E> s2) {  
	Set<E> result = new HashSet<>(s1);  
	result.addAll(s2);  
	return result;  
}  

// 향상된 유연성을 확인해주는 맛보기 프로그램 (185쪽)  
public static void main(String[] args) {  
	Set<Integer> integers = new HashSet<>();  
	integers.add(1);  
	integers.add(3);  
	integers.add(5);  

	Set<Double> doubles = new HashSet<>();  
	doubles.add(2.0);  
	doubles.add(4.0);  
	doubles.add(6.0);  

	Set<Number> numbers = union(integers, doubles);  

//      // 코드 31-6 자바 7까지는 명시적 타입 인수를 사용해야 한다. (186쪽)  
//      Set<Number> numbers = Union.<Number>union(integers, doubles);  

	System.out.println(numbers);  
}
```

제대로만 사용하면 클래스 사용자는 와일드카드 타입이 쓰였다는 사실조차 의식하지 못함.

**클래스 사용자가 와일드카드 타입을 신경써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.**

```java
// public static <E extends Comparable<E>> E max(  
//        List<E> list) {
public static <E extends Comparable<? super E>> E max(  
        List<? extends E> list) {  
    if (list.isEmpty())  
        throw new IllegalArgumentException("빈 리스트");  
  
    E result = null;  
    for (E e : list)  
        if (result == null || e.compareTo(result) > 0)  
            result = e;  
  
    return result;  
}  
  
public static void main(String[] args) {  
    List<String> argList = Arrays.asList("a", "b", "c", "g", "d");  
    System.out.println(max(argList));  
}
```
위 코드는 PECS 공식을 2번 적용한 코드입니다.

- 입력 매개변수
	- E 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? extends E>`로 수정.
- 타입 매개변수
	- E가 `Comparable<E>`를 확장.
	- `Comparable<E>`는 E 인스턴스를 소비.
	- 그래서 매개변수화 타입 `Comparable<E>`를 한정적 와일드카드 타입인 `Comparable<? super E>`로 대체.

`Comparable`이나 `Comparator`는 언제나 소비자이므로 `<? super E>`를 사용하는 편이 낫다고 합니다.

이렇게까지 복잡하게 만들만한 가치가 있는가? '그렇다'라고 합니다.

```java
List<ScheduledFuture<?>> scheduledFuters = ....;
```
수정 전 max는 이 리스트를 처리할 수 없음.
`ScheduledFuture는 Comparable<ScheduledFuture>`를 구현하지 않았기 때문.

##### 관계
`ScheduledFuture -> Deleayd -> Comparable` 
`Comparable`을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드 카드가 필요.

### 타입 매개변수와 와일드카드
타입 매개변수와 와일드카드에는 공통되는 부분이 많아서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.

```java
    private static <E> void swap(List<E> list, int i, int j) {  
    private static void swap(List<?> list, int i, int j) {
```
두 메서드 중에 public API라면 두번째가 낫다.

#### 기본규칙
- 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.
- 비한정적 타입 매개변수면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 타입 와일드카드로.

```java
private static void swap(List<?> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));  
}
```
이 코드에서는 에러가 발생하는데 `List<?>`에는 null 외에는 어떤 값도 넣을 수 없다고 합니다.

형변환이나 리스트의 로 타입을 사용하지 않고도 해결하는 방법이 있는데  
바로 와일드카드 타입의 실제 타입을 알려주는 메서드를 `private`도우미 메서드로 따로 작성하여 활용하는 방법.

```java
    public static void swap(List<?> list, int i, int j) {  
        swapHelper(list, i, j);  
    }  
  
    // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드  
    private static <E> void swapHelper(List<E> list, int i, int j) {  
        list.set(i, list.set(j, list.get(i)));  
    }
```

조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.
그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다.
PECS 공식을 기억!
