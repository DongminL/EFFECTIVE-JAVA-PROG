## 정수 열거 패턴

자바에서 열거 타입을 지원하기 전에는 정수 상수를 한 묶음 선언해서 사용하곤 했다.

```java
// 코드 34-1
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

- 이같은 방식은 단점 투성이다.
    - 타입 안전을 보장할 수 없다.
        - APPLE_FUJI와 ORANGE_NAVEL은 서로 다른 개념임에도, 같은 값을 가진다.
    - 표현력이 떨어진다.
        - 별도 이름공간을 지원하지 않기에, 접두어를 명시해주어야 한다.
    - 평범한 상수의 나열이기에, 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.
        - 이것은 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 함을 의미한다.
    - 문자열 출력이 까다롭다.
    - 같은 정수 열거 그룹에 속한 모든 상수를 순회하기도 어렵다.

## 문자열 열거 패턴

정수 대신 문자열 상수를 사용하는 패턴을 의미한다. 이는, 정수 열거 패턴보다 더 나쁘다.

- 이유
    - 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하는 프로그래머가 존재할 수 있다.
        - 오타를 유발할 수 있음.
    - 문자열 비교에 따른 성능 저하도 야기한다.

## enum type

- 이같은 단점을 보완해줄 enum 타입이 Java 1.5부터 도입되었다.

### 자바 열거 타입의 특징

- 열거 타입 자체는 클래스이다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어서 public static final 필드로 공개한다.
- 외부에서 생성자에 접근할 수 없다.
- 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재함이 보장된다.

### 장점

- 컴파일타임에 타입 안전성을 제공한다.
    
    ```java
    // 코드 34-2
    public enum Apple { FUJI, PIPPIN, GRANNY_SMITH}
    public enum Orange { NAVEL, TEMPLE, BLOOD}
    ```
    
    - Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면, 건네받은 참조는 Apple의 세 가지 값 중 하나임이 확실하다.
    - 다른 타입의 값을 넘기려 하면 컴파일 오류가 난다.
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
    - 기본적으로 상수 이름을 반환, 원한다면 재정의
- 같은 정수 열거 그룹에 속한 모든 상수를 순회하기 쉽다.
    - 정의된 상수들의 값을 배열에 담아 반환하는 values 메서드를 제공한다.
- 만약 상수 하나를 제거한다면?
    - 클라이언트 프로그램을 다시 컴파일하면, 제거된 상수를 참조하는 쪽에서 컴파일 오류가 발생하여, 사전에 식별할 수 있다.
- 임의의 필드나 메서드를 추가할 수 있고, 임의의 인터페이스를 구현하게 할 수도 있다.
    
    ```java
    // 코드 34-3 데이터와 메서드를 갖는 열거 타입 (211쪽)
    public enum Planet {
        MERCURY(3.302e+23, 2.439e6),
        VENUS  (4.869e+24, 6.052e6),
        EARTH  (5.975e+24, 6.378e6),
        MARS   (6.419e+23, 3.393e6),
        JUPITER(1.899e+27, 7.149e7),
        SATURN (5.685e+26, 6.027e7),
        URANUS (8.683e+25, 2.556e7),
        NEPTUNE(1.024e+26, 2.477e7);
    
        private final double mass;           // 질량(단위: 킬로그램)
        private final double radius;         // 반지름(단위: 미터)
        private final double surfaceGravity; // 표면중력(단위: m / s^2)
    
        // 중력상수(단위: m^3 / kg s^2)
        private static final double G = 6.67300E-11;
    
        // 생성자
        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
            surfaceGravity = G * mass / (radius * radius);
        }
    
        public double mass()           { return mass; }
        public double radius()         { return radius; }
        public double surfaceGravity() { return surfaceGravity; }
    
        public double surfaceWeight(double mass) {
            return mass * surfaceGravity;  // F = ma
        }
    }
    ```
    
    - 단순하지만, 놀랍도록 강력하다.

## 필드, 메서드, 멤버 클래스

- 필드
    - 열거 타입은 근본적으로 불변이라, 모든 필드는 final이어야 한다.
    - 필드는 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는 게 낫다.
- 메서드
    - 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현하라.
- 멤버 클래스
    - 특정 톱레벨 클래스에서만 쓰이는 열거 타입은 해당 클래스의 멤버 클래스로 만들어라.

## 상수의 더 다양한 기능 제공

- 상수마다 동작이 달라져야 하는 상황
    1. swtich문 이용 (bad)
        
        ```java
        // 코드 34-4
        public enum Operation {
            PLUS,MINUS,TIMES,DIVDE;
        
            public double apply(double x, double y) {
                switch (this) {
                    case PLUS:
                        return x + y;
                    case MINUS:
                        return x - y;
                    case TIMES:
                        return x * y;
                    case DIVDE:
                        return x / y;
                }
                
                throw new AssertionError("알 수 없는 연산:" + this);
            }
        }
        ```
        
        - 동작은 하지만, 아름답지 못하다.
            - 어쩔 수 없이 `throw new AssertionError("알 수 없는 연산:" + this);` 구문을 작성해 주어야 하기에.
        - 또한, 깨지기 쉽다.
            - 새로운 상수를 추가하면, 해당 case 문도 수정해야 한다.
    2. 상수별 메서드 구현 (good)
        
        ```java
        public enum Operation {
            PLUS {
                @Override
                public double apply(double x, double y) {
                    return x + y;
                }
            },MINUS {
                @Override
                public double apply(double x, double y) {
                    return x - y;
                }
            },TIMES {
                @Override
                public double apply(double x, double y) {
                    return x * y;
                }
            },DIVDE {
                @Override
                public double apply(double x, double y) {
                    return x / y;
                }
            };
            
            public abstract double apply(double x, double y);
        }
        ```
        
        - 추상 메서드를 선언하고, 각 상수별 클래스 몸체에서 재정의하는 방식이 가능하다.
        - 장점
            - 재정의하지 않으면, 컴파일 오류 발생
- valueOf(String) 메서드
    - 해당 메서드는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 것으로, 자동 생성된다.

## 열거 타입에서의 toString 재정의

열거 타입의 toString 메서드를 재정의하려거든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하라.

```java
// 코드 34-7 열거 타입용 fromString 메서드 구현하기 (216쪽)
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
                toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

눈여겨볼 점

1. Operation 상수가 stringToEnum에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때이다.
    - 열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없기에, 위처럼 초기화해야 한다.
2. fromString이 Optional<Operation>을 반환하는 점.
    - 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 전달하는 목적

## 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.

```java
// 코드 34-8
public enum PayrollDay {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;
        int overtimePay;

        switch (this) {
            // 주말
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            // 주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
```

- 간결하지만, 관리 관점에서는 위험한 코드이다.
    - 휴가와 같은 새로운 값을 추가한다고 해보자.
        - 그 값을 처리하는 case 문을 쌍으로 넣어주는 걸 깜빡했다면 프로그램은 말끔히 컴파일되지만, 평일과 똑같은 임금을 받게 된다.
- 이처럼 열거 타입 상수 일부가 같은 동작을 공유한다면, 전략 열거 타입 패턴을 고려하라.
    
    ```java
    // 코드 34-9 전략 열거 타입 패턴 (218-219쪽)
    enum PayrollDay {
        MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
        THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
        SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    
        private final PayType payType;
    
        PayrollDay(PayType payType) { this.payType = payType; }
        
        int pay(int minutesWorked, int payRate) {
            return payType.pay(minutesWorked, payRate);
        }
    
        // 전략 열거 타입
        enum PayType {
            WEEKDAY {
                int overtimePay(int minsWorked, int payRate) {
                    return minsWorked <= MINS_PER_SHIFT ? 0 :
                            (minsWorked - MINS_PER_SHIFT) * payRate / 2;
                }
            },
            WEEKEND {
                int overtimePay(int minsWorked, int payRate) {
                    return minsWorked * payRate / 2;
                }
            };
    
            abstract int overtimePay(int mins, int payRate);
            private static final int MINS_PER_SHIFT = 8 * 60;
    
            int pay(int minsWorked, int payRate) {
                int basePay = minsWorked * payRate;
                return basePay + overtimePay(minsWorked, payRate);
            }
        }
    
        public static void main(String[] args) {
            for (PayrollDay day : values())
                System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
        }
    }
    ```
    
    - 잔업수당 계산을 PayType이라는 열거 타입으로 옮기고, PayrollDay 열거 타입의 생성자에서 적당한 것을 선택하게 한다.
    - 그럼, PayrollDay 열거 타입은 잔업수당 계산을 전략 열거 타입에 위임하여, 별도의 구현이 필요없어진다.
    - 장점
        - 안전성 ↑
        - 유연성 ↑
    - 단점
        - 복잡도 ↑

## 그럼, switch는 언제 사용?

- 위에서 확인한 것처럼, 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다.
- 하지만, 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 좋은 선택이 될 수 있다.
    
    ```java
    // 코드 34-10 switch 문을 이용해 원래 열거 타입에 없는 기능을 수행한다. (219쪽)
    public class Inverse {
        public static Operation inverse(Operation op) {
            switch(op) {
                case PLUS:   return Operation.MINUS;
                case MINUS:  return Operation.PLUS;
                case TIMES:  return Operation.DIVIDE;
                case DIVIDE: return Operation.TIMES;
    
                default:  throw new AssertionError("Unknown op: " + op);
            }
        }
    
        public static void main(String[] args) {
            double x = Double.parseDouble(args[0]);
            double y = Double.parseDouble(args[1]);
            for (Operation op : Operation.values()) {
                Operation invOp = inverse(op);
                System.out.printf("%f %s %f %s %f = %f%n",
                        x, op, y, invOp, y, invOp.apply(op.apply(x, y), y));
            }
        }
    }
    ```
    

## 열거 타입은 언제 사용?

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 사용할 것.
    - 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없음.
