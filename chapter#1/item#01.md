# 생성자 대신 정적 팩터리 메서드를 고려하라
생성자 대부분의 상황에서 적절할 수 있지만 정적 팩토리 메서드가 유효할 때가 있습니다.

## 장점
### 첫 번째. 이름을 가질 수 있다.

![image](https://github.com/user-attachments/assets/dc0a31af-873a-425b-91e4-7e53e1a57f22)

위와 같은 방식으로는 생성자를 만들 수 없습니다.

```java
public Order(Product product, boolean urgent) {  
    this.urgent = urgent;  
    this.product = product;  
}  
// 이렇게는 가능하지만 생성자는 이름을 지을 수 없어서 위와 같이 코드를 짜면 알기가 어렵다.
```
이렇게는 가능하지만

```java
public static Order primeOrder(Product product, boolean prime) {  
    Order order = new Order();  
    order.prime = prime;  
    order.product = product;  
    return order;  
}  
  
public static Order urgentOrder(Product product, boolean urgent) {  
    Order order = new Order();  
    order.urgent = urgent;  
    order.product = product;  
    return order;  
}
```

### 두 번째, 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
기본 생성자가 있는 경우엔 객체 생성을 얼마든지 사용하는 쪽에서 만들 수 있습니다.

기본 생성자가 public인 경우엔 객체를 생성하고 해시 코드 값을 비교하면 매번 다른 값이 나오는 걸 볼 수 있습니다.
```java
public class Settings {  
    private boolean useAutoSteering;  
    private boolean useABS;  
    private Difficulty difficulty;  
  
    private Settings() {  
    }  
}
```
**하지만 기본 생성자를 private 으로 하게 되면??**
다른 곳에서 기본 생성자를 호출하지 못하게 됩니다. (객체 생성을 자신이 컨트롤 하겠다는 의미)

```java
private static final Settings SETTINGS = new Settings();  
public static Settings newInstance() {  
    return SETTINGS;  
}
```
위와 같은 코드로 객체를 생성하면 늘 같은 해시 코드 값이 나오는걸 볼 수 있습니다.
![image](https://github.com/user-attachments/assets/5efb6250-ae8f-45d7-a275-d3bd0404520e)

이런 클래스를 인스턴스-통제 클래스라 합니다.

>반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩토리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다.
>인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴화 불가로 만들 수도 있다.

**플라이웨이트 패턴**
https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-Flyweight-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90

### 세 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
```java
public interface HelloService {  
}


///////////

public class HelloServiceFactory {  
    public static HelloService of(String language) {  
        if(language.equals("ko")) {  
            return new KoreanHelloService();  
        }else {  
            return new EngHelloService();  
        }  
    }  
}
// 네번째 장점도 여기에서 보여집니다. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
// 자바 8 이후로 가능해짐.

////////////

HelloService ko = HelloServiceFactory.of("ko");
```
API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있습니다.

```java.util.Collections``` 클래스는 생성자가 private이기 때문에 인스턴스화 불가 클래스.

컬렉션 프레임워크는 45개의 클래스를 공개하지 않기 때문에 클라이언트가 API를 더욱 쉽고 간편하게 사용할 수 있습니다.

### 네 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
>반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {  
    Enum<?>[] universe = getUniverse(elementType);  
    if (universe == null)  
        throw new ClassCastException(elementType + " not an enum");  
  
    if (universe.length <= 64)  
        return new RegularEnumSet<>(elementType, universe);  
    else  
        return new JumboEnumSet<>(elementType, universe);  
}
```
클라이언트는 이 두 클래스의 존재를 모르고 알 필요도 없습니다. 단지 반환 타입의 하위 타입이기만 하면 됩니다.

### 다섯 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다고 합니다.
https://kkang-joo.tistory.com/46

서비스 제공자 프레임워크는 3개의 핵심 컴포넌트로 이루어진다.
1. 서비스 인터페이스
2. 제공자 등록 API
3. 서비스 접근 API
서비스 제공자 프레임워크에서의 제공자는 서비스의 구현체. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.

자바 6 이후부턴 ServiceLoader 라는 범용 서비스 제공자 프레임워크가 제고오디어 프레임워크를 직접 만들 필요가 거의 없어졌다고 합니다.

## 단점
### 첫 번째, 상속을 하려면 Public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
컬렉션 프레임워크를 상속할 수 없다는 이야기라고 생각하면 된다.

### 두 번째, 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스르 인스턴스화할 방법을 알아내야 한다. 

메서드 이름도 알려진 규약을 따라 짓는 식으로 문제를 완화해줘야 함.
