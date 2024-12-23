`float`와 `double`타입은 과학과 공학 계산용으로 설계되었다.
이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 **근사치**로 계산하도록 세심하게 설계되었다.

따라서 정확한 결과가 필요할 때는 사용하면 안된다!

`float`와 `double`타입은 특히 금융 관련 계산과는 맞지 않는다.

`System.out.println(1.03 - 0.42);`를 계산을 한다면 결과는  `0.6100000000000001`이 나오게 된다.

결괏값을 출력하기 전에 반올림하면 해결되리라 생각할지 모르지만, 반올림을 해도 틀린 답이 나올 수 있다.


### double
다음 예시 코드를 보면
```java
public static void main(String[] args) {  
    double funds = 1.00;  
    int itemsBought = 0;  
    for (double price = 0.10; funds >= price; price += 0.10) {  
        funds -= price;  
        itemsBought++;  
    }  
    System.out.println(itemsBought + "개 구입");  // 3개 구입
    System.out.println("잔돈(달러): " + funds); // 잔돈(달러): 0.3999999999999999
}
```
잘못된 결과가 나오게 된다.

정확한 결과를 계산하기 위해선 `BigDecimal`혹은 `int`, `long`을 사용해야 한다.

### BigDecimal
```java
public static void main(String[] args) {  
    final BigDecimal TEN_CENTS = new BigDecimal(".10");  
  
    int itemsBought = 0;  
    BigDecimal funds = new BigDecimal("1.00");  
    for (BigDecimal price = TEN_CENTS;  
         funds.compareTo(price) >= 0;  
         price = price.add(TEN_CENTS)) {  
        funds = funds.subtract(price);  
        itemsBought++;  
    }  
    System.out.println(itemsBought + "개 구입"); // 4개 구입
    System.out.println("잔돈(달러): " + funds); // 잔돈(달러): 0.00
}
```
올바른 결과가 나온다.

하지만 `BigDecimal`에는 단점이 두 가지 있다.

1. 기본 타입보다 쓰기가 훨씬 불편하다.
2. 훨씬 느리다.

`BigDecimal`의 대안으로 `int`, `long`타입을 쓸 수도 있다.

### int, long
```java
public static void main(String[] args) {
	int itemsBought = 0;
	int funds = 100;
	for (int price = 10; funds >= price; price += 10) {
		funds -= price;
		itemsBought++;
	}
	System.out.println(itemsBought + "개 구입"); // 4개 구입
	System.out.println("잔돈(센트): " + funds); // 잔돈(센트): 0
}
```
소수점을 전부 올려서 계산을 수행하면 해결된다. (다룰 수 있는 값의 크기가 제한적임.)

## 정리
- 정확한 답이 필요한 계산에는 `float`과 `double`은 피하라.
- 소수점 추적은 코딩 시의 불편함이나 성능 저하를 신경 쓰지 않겠다면 `BigDecimal`을 사용하라.
	- 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리함.
- 성능이 중요하고 숫자나 너무 크지 않다면 `int`, `long`을 사용하라.
- 아홉 자리 십진수로 표현할 수 있다면 `int`
- 열여덟 자리 십진수로 표현할 수 있다면 `long`
- 열여덟 자리를 넘어가면 `BigDecimal`을 사용해야 한다.
