# Item 86. Serializable을 구현할지는 신중히 결정하라

## Serializable 구현 후 수정은 어렵다

**어떤 클래스의 인스턴스를 직렬화할 수 있게 하려면 클래스 선언에 `implements Serializable`만 덧붙이면 된다**. 쉽게 적용할 수 있어 **신경 쓸 게 없다고 오해할 수 있지만, 엄청 복잡하고 아주 값비싼 일**이다.

**`Serializable`을 구현하면 릴리스한 뒤에는 수정하기 어렵다**. 클래스가 `Serializable`을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화 형태)도 하나의 **공개 API가 되는 것**이기 때문이다. 널리 퍼진다면 그 직렬화 형태도 **영원히 지원**해야 한다. 만약 자바의 기본 방식을 사용하면, 직렬화 형태는 최소 적용 당시 클래스의 내부 구현 방식에 영원히 종속된다.

**기본 직렬화 형태에서는 클래스의 `private`과 `package-private` 인스턴스 필드들도 API로 공개**된다. 즉, **캡슐화가 깨져** 필드로의 접근을 최대한 막아 정보를 은닉하라는 조언도 무력화된다.

뒤늦게 클래스 내부 구현 방식을 수정하면 원래 직렬화 형태와 달라지게 된다. 직렬화와 역직렬화의 버전이 달라지게 되면, 실패를 볼 수 있다. **직렬화 가능 클래스를 만들고자 한다면, 길게 보고 감당할 수 있을 만큼 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다**.

### 직렬화가 클래스 개선을 방해하는 예 - 직렬 버전 UID
---

**모든 직렬화된 클래스는 고유 식별 번호를 부여**받는다. `static final long serialVersionUID` 필드로 번호를 명시하지 않으면 시스템이 런타임에 암호 해시 함수(SHA-1)를 적용해 자동으로 클래스 안에 생성해 넣는다. 그러나 **자동 생성 값에 의존하고 나중에 클래스를 수정하면, 쉽게 호환성이 깨져버려 런타임에 `InvalidClassException`이 발생할 것이다**.

<br>

## 버그와 보안 구멍이 생길 위험이 높아진다

**`Serializable` 구현의 두 번재 문제는 버그와 보안 구멍이 생길 위험이 높아진다는 점**이다. 객체는 생성자를 사용해 만드는 것이 기본이다. 따라서 **직렬화**는 언어의 기본 메커니즘을 **우회하는 객체 생성 기법**이다. **기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다**는 뜻이다.

<br>

## 신버전 릴리스 시, 테스트할 것이 늘어난다

**`Serializable` 구현의 세 번째 문제는 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다는 점이다**. 직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그리고 그 반대도 가능한지를 검사해야 한다. 양방향 직렬화/역직렬화가 모두 성공하고, 원래의 객체를 충실히 복제해내는지를 반드시 확인해야 한다.

클래스를 처음 제작할 때 커스텀 직렬화 형태를 잘 설계했다면 이러한 테스트 부담을 줄일 수 있다.

<br>

## Serializable 구현 여부는 가볍게 결정할 사안이 아니다

**`Serializable` 구현 여부는 가볍게 결정할 사안이 아니다**. 그러나 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스라면 어쩔 수 없이 구현해야 한다. 또한 `Serializable`을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스도 마찬가지다.

`Serializable` 구현에 따르는 비용이 적지 않으니, 클래스를 설계할 때마다 그 이득과 비용을 잘 비교해봐야 한다.

**역사적으로 `BigInteger`와 `Integer` 같은 '값' 클래스와 컬렉션 클래스들은 `Serializable`을 구현하고, 스레드 풀처럼 '동작'하는 객체를 표현하는 클래스들은 `Serializable`을 구현하지 않았다**.

<br>

## 상속용은 클래스는 구현하지말고, 인터페이스는 확장하지 말자

**상속용으로 설계된 클래스는 대부분 `Serializable`을 구현하면 안 되고, 인터페이스도 대부분 `Serializable`을 확장해서는 안 된다**. 이 규칙을 따르지 않는다면, 그런 클래스나 인터페이스를 확장하고 구현하는 사람에게 큰 부담을 지우게 되는 것이다.

### 어겨야 하는 상황 - Throwable
---

그러나 이 규칙을 어겨야 하는 상황이 있다. **`Serializable`을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 다른 방도가 없을 것이다**.

상속용으로 설계된 클래스 중 `Serializable`을 구현한 예로는 `Throwable`과 `Component`가 있다. `Throwable`은 서버가 RMI를 통해 클라이언트로 예외를 보내기 위해 구현했다. `Component`는 GUI를 전송하고 저장하고 복원하기 위해 구현했지만, `Swing`과 `AWT`를 많이 사용하던 시절에도 현업에서는 이런 용도로는 거의 사용하지 않았다.

> RMI : 분산되어 존재하는 객체 간의 메시지 전송(메소드를 호출하는 것 포함)을 가능하게 하는 프로토콜

<br>

## finalize 메서드는 재정의하지 못하게 하자

**클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능하다면** 주의할 점이 있다. **인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 반드시 하위 클래스에서 `finalize` 메서드를 재정의하지 못하게 해야 한다**. 따라서 **`finalize` 메서드를 자신이 재정의하면서 필드는 `final`로 선언**하면 된다. **이렇게 해두지 않으면 `finalizer` 공격을 당할 수 있다**.

인스턴스 필드 중 기본값으로 초기화되면 위배되는 불변식이 있다면 클래스에 다음과 같은 `readObjectNoData` 메서드를 반드시 추가해야 한다.

``` java
// 코드 86-1 상태가 있고, 확장 가능하고, 직렬화 가능한 클래스용 readObjectNoData 메서드
private void readObjectNoData() throws InvalidObjectException {
    throw new InvalidObjectException("스트림 데이터가 필요합니다");
}
```

기존의 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드다.

<br>

## `Serializable`을 구현하지 않기로 할 때 주의점

`Serializable`을 구현하지 않기로 할 때는 한 가지만 주의하면 된다. 상속용 클래스인데 직렬화를 지원하지 않으면 그 하위 클래스에서 직렬화할 때 부담이 늘어난다.

보통 **이런 클래스를 역직렬화 하기 위해서는 그 상위 클래스는 매개변수가 없는 생성자를 제공**해야 한다. **이런 생성자를 제공하지 않으면 하위 클래스에서**는 어쩔 수 없이 **직렬화 프록시 패턴을 사용**해야 한다.

<br>

## 내부 클래스는 직렬화를 구현하지 말자

내부 클래스에는 외부 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다. 내부 클래스에 대한 기본 직렬화 형태는 분명하지가 않다. 그러나 **정적 멤버 클래스는 `Serializable`을 구현해도 된다**.

<br>

## 정리

- Serializable은 구현한다고 선언하기는 아주 쉽지만, 그것은 눈속임일 뿐이다.

- 보호된 환경에서만 쓰일 클래스가 아니라면 Serializable 구현은 아주 신중히 하자.

- 상속할 수 있는 클래스라면 주의사항이 더욱 많다.
