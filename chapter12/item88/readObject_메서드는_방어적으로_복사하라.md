`readObject` 메서드는 역직렬화 시 객체의 상태를 복원하는 역할을 하며, 이는 사실상 또 다른 `public` 생성자와 같다.  
-> `readObject`는 매개변수로 바이트 스트림을 받는 생성자.

보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다. 하지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다.

```java
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```
위 코드에서 생성자와 접근자에서 방어적 복사를 통해 불변성을 유지.

그러나 `Seializable`을 구현하고 `readObject` 메서드를 적절히 작성하지 않으면 역직렬화 시 불변성이 깨질 수 있다.

책에 있는 `BogusPeriod`를 실행해보니까 `Caused by: java.lang.ClassNotFoundException: Period` 오류만 떠서 제 컴퓨터에서 실행해보진 못했습니다...

이 문제를 고치려면 `Period`의 `readObject` 메서드가 `defaultReadObject`를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다. 
```java
// 유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다!
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 불변식을 만족하는지 검사.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
이상의 작업으로 공격자가 허용되지 않는 `Period` 인스턴스를 생성하는 일을 막을 수 있지만, 아직도 미묘한 문제가 하나 숨어 있다.

공격자는 `ObjectInputStream`에서 `Period`인스턴스를 읽은 후 스트림 끝에 추가된 이 '악의적인 객체 참조'를 읽어 `Period` 객체의 내부정보를 얻을 수 있다.

```java
public class MutablePeriod {
    public final Period period;
    public final Date start; // 외부 접근 불가
    public final Date end; // 외부 접근 불가

    public MutablePeriod() throws Exception {
        // Period 인스턴스를 직렬화된 바이트 배열로 만듦
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream out = new ObjectOutputStream(bos);

        out.writeObject(new Period(new Date(), new Date()));
        byte[] ref = {0x71, 0, 0x7e, 0, 5};
        bos.write(ref); // start field
        ref[4] = 4;
        bos.write(ref); // end field
        out.close();

        // Period 객체 역직렬화
        ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
        period = (Period) in.readObject();
        start = (Date) in.readObject();
        end = (Date) in.readObject();
        in.close();
    }

    public static void main(String[] args) throws Exception {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;
        
        pEnd.setYear(78);
        System.out.println(p);

		pEnd.setYear(69);
        System.out.println(p);
    }
}
```
이 예에서 `Period` 인스턴스는 불변식을 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있었다.

이 문제의 근원은 `Period`의 `readObject` 메서드가 방어적 복사를 충분히 하지 않은 데 있다.  
객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.

`readObject`에서는 불변 클래스 안의 모든 `private` 가변 요소를 방어적으로 복사해야 한다.
```java
// 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사합니다.
	start = new Date(start.getTime());
	end = new Date(end.getTime());

    // 불변식을 만족하는지 검사합니다.
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```
방어적 복사를 유효성 검사보다 앞서 수행. `Date`의 `clone`메서드는 사용하지 않았음에 주목.

두 조치 모두 `Period`를 공격으로부터 보하는 데 필요하다.

또한 `final`필드는 방어적 복사가 불가능하니 주의!

그래서 이 `readObject`메서드를 사용하려면 `start`와 `end`필드에서 `final`한정자를 제거해야 한다.

`transient`필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 `public` 생성자를 추가해도 괜찮은가? 답이 "아니오"라면 커스텀 `readObject`메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행해야 한다.  
혹은 직렬화 프록시 패턴(아이템 90)을 사용하는 방법도 이싿. 이 패턴은 역직렬화를 안전하게 만드는 데 필요한 노력을 상당히 경감해주므로 적극 권장하는 바다.

`final`이 아닌 직렬화 가능 클래스라면 `readObject`와 생성자의 공통점이 하나 더 있다. 마치 생성자처럼 `readObject`메서드도 재정의 가능 메서드를 호출해서는 안된다. 이 규칙을 어겼는데 해당 메서드가 재정의 되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행된다.

## 정리
- `private`이어야 하는 객체 참조 필드은 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
- 모든 불변식을 검사하여 어긋나는 게 발견되면 `InvalidObjectException`을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 `ObjectInputValidation`인터페이스를 사용하라.
- 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.
