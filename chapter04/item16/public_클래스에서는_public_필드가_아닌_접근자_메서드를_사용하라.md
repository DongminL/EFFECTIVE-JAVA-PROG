
## 퇴보한 클래스는 public 이어서는 안 된다!

```java
class Point {
	public double x;
	public double y;
}
```

멤버 변수가 public 으로 선언되어 있어, 외부에서 직접 접근 및 수정이 가능하기 때문에 캡슐화를 위반한다.

```java
/* public */ class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

public 클래스라면 접근자와 변경자 메서드를 활용해 데이터를 캡슐화 하는 것이 좋다.

## packege-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 하등의 문제가 없다

접근 범위가 제한적이기 때문에 데이터 필드를 public으로 노출하더라도 큰 문제가 발생하지 않기 때문.

private 중첩 클래스는 외부 클래스에서만 접근할 수 있으며, 그 외부에서는 접근이 불가능 합니다.  
따라서, 이러한 중첩 클래스의 필드를 public으로 선언하더라도 외부 클래스에서만 해당 필드에 접근할 수 있으므로, 데이터 노출에 대한 위험이 적습니다.

### 의문점
>이 방식이 클래스 선언 면에서나 이를 사용하는 클라이어튼 코드 면에서나 접근자 방식보다 훨씬 깔끔

```java
Point point = new Point();  
point.x = 1;  
point.y = 2;
```
- 필드를 public 으로 선언하면, 해당 필드에 직접 접근할 수 있어 코드가 간결.
- 접근자 메서드를 사용하지 않으면 클래스 내에 추가적인 메서드를 정의할 필요가 없어, 클래스의 복잡성이 줄어듬.
- 내부 구현을 단순화하여 유지보수성을 높일 수 있음.
- 필드를 final로 선언하고 public으로 공개하면, 해당 필드가 불변임을 명확하게 나타낼 수 있음.
- 메소드 호출 오버헤드 감소.

그러나 이러한 접근은 클래스의 **사용 범위가 제한적일 때에만** 적용하는 것이 좋다고 합니다.

일반적으로는 캡슐화 원칙을 준수하여 필드를 private으로 선언하고, 필요한 경우 접근자 메서드를 제공하는 것이 바람직합니다.


## 타산지석 삼을 클래스

![image](https://github.com/user-attachments/assets/135d3937-a80b-4312-b868-59c429de42a7)


![image](https://github.com/user-attachments/assets/9ca182f1-6b37-46dc-bc2d-0b770ee232d8)


## 불변 필드를 노출한 public 클래스
```java
// 좋은 클래스인가?
public final class Time {  
    private static final int HOURS_PER_DAY    = 24;  
    private static final int MINUTES_PER_HOUR = 60;  
  
    public final int hour;  
    public final int minute;  
  
    public Time(int hour, int minute) {  
        if (hour < 0 || hour >= HOURS_PER_DAY)  
            throw new IllegalArgumentException("Hour: " + hour);  
        if (minute < 0 || minute >= MINUTES_PER_HOUR)  
            throw new IllegalArgumentException("Min: " + minute);  
        this.hour = hour;  
        this.minute = minute;  
    }  
  
    // 나머지 코드 생략  
}
```

위 코드는 hour와 minute 필드가 public으로 선언되어 있어 외부에서 직접 접근이 가능합니다.

이는 불변 객체의 설계 원칙에 어긋날 수 있습니다.  
일반적으로 불변 객체의 필드는 private으로 선언하고, 해당 값을 반환하는 **접근자 메서드(getter)** 를 제공하는 것이 권장됩니다.
