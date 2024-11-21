equals 메서드는 재정의가 쉬워 보이지만, 곳곳에 함정이 숨어있다.

문제를 회피하는 가장 좋은 방법은 재정의를 하지 않는 것이다.

# 재정의를 하지 않는 것이 최선인 상황

1. 각 인스턴스가 본질적으로 고유한 상황
    - 값을 표현하는 게 아닌, 동작 객체를 표현하는 클래스 (ex. Thread)
2. 인스턴스의 ‘논리적 동치성(logical equality)’을 검사할 일이 없는 상황
    
    > 논리적 동치?
    같은 객체인지가 아닌, 같은 내용인지 비교하는 것.
    > 
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 상황
    - java.util.AbstractSet
        
        ```java
        public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
            public boolean equals(Object o) {
        			 // 생략
            }
        
            public int hashCode() {
           		 // 생략
            }
        }
        ```
        
        - 대부분의 Set 구현체는 AbstractSet에서 구현한 equals를 상속받아 쓴다.
            
            ```java
            public class HashMap<K,V> extends AbstractMap<K,V>
                implements Map<K,V>, Cloneable, Serializable {
                ...
            }
            ```
            
    - List, Map도 마찬가지. 상속받아 그대로 쓴다.
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 상황
    - public의 경우에는 다양하게 활용될 가능성이 있으나, private이나 package-private의 경우 equals 메서드를 사용할 일이 없다면 재정의할 필요가 없다.
    - 만약 호출조차 막고싶다면 하기의 코드처럼 작성할 수 있다.
        
        ```java
        @Override
        public boolean equals(Object o) {
        	throw new AssertionError(); // 호출 금지!
        }
        ```
        

# 재정의를 해야 하는 상황

객체 식별성(= 두 객체가 물리적으로 같은가)이 아니라, 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때.

## equals 메서드 재정의 시 따라야 하는 일반 규약

### 왜 지켜야 하는가? 🤔

클래스의 인스턴스는 다른 곳으로 빈번히 전달되며, 수많은 클래스들은 전달받은 객체가 equals 규약을 지킨다고 가정하고 동작한다.

따라서, 규약을 어기게 되면 그 객체를 사용하는 다른 객체들이 어떻게 반응할 지 예상할 수 없다.

### equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

- 반사성(reflexivity)
    
    > null이 아닌 모든 참조값 x에 대해 x.equals(x)는 true다.
    > 
- 대칭성(symmetry)
    
    > null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
    > 
- 추이성(transitivity)
    
    > null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
    > 
- 일관성(consistency)
    
    > null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
    > 
    - 즉, equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다. (ex. 네트워크)
- null-아님
    
    > null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
    > 
    - NullPointerException을 던지는 코드는 종종 있을 수 있는데, 이것 또한 일반 규약을 위반하는 것이다.

### 첫번째 예제(대칭성 위반)

```java
import java.util.Objects;

public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    // 대칭성 위배!
    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    
    public static void main(String[] args) {
        CaseInsensitiveString a = new CaseInsensitiveString("Polish");
        String b = "polish";
        System.out.println(a.equals(b)); // true
        System.out.println(b.equals(a)); // false
    }
}
```

### 문제 해결

CaseInsensitiveString의 equals를 String과도 연동하겠다는 꿈을 버려라.

```java
@Override public boolean equals(Object o) {
    return (o instanceof CaseInsensitiveString) && s.equalsIgnoreCase(
        ((CaseInsensitiveString) o).s);
}
```

### 두번째 예제(대칭성, 추이성 위반)

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    @Override public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

- 현재 상태에서는 색상 정보는 무시한 채 비교를 수행한다.
- 이를 개선하고자, equals를 다음과 같이 재정의해보자.
    
    ```java
    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
    
        return super.equals(o) && ((ColorPoint) o).color == color;
    }
    ```
    
    - 그럼 다음과 같이 대칭성을 위배하게 된다.
        
        ```java
        Point p = new Point(1, 2);
        ColorPoint cp = new ColorPoint(1, 2, Color.RED);
        
        p.equals(cp); // true
        cp.equals(p); // false
        ```
        
    - 이를 해결하고자, ColorPoint의 equals에서 Point와 비교할 때는 색상을 무시하도록 해보자.
        
        ```java
        @Override public boolean equals(Object o) {
            if (!(o instanceof Point)) {
                return false;
            }
        
            // o가 일반 Point면 색상을 무시하고 비교한다.
            if (!(o instanceof ColorPoint)) {
                return o.equals(this);
            }
        
            // o가 ColorPoint면 색상까지 비교한다.
            return super.equals(o) && ((ColorPoint) o).color == color;
        }
        ```
        
        - 대칭성은 지켰으나, 추이성을 위배한다.
            
            ```java
            ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
            Point p2 = new Point(1, 2);
            ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
            
            p1.equals(p2); // true
            p2.equals(p3); // true
            p1.equals(p3); // false
            ```
            
        - 또한, 이 방식은 무한 재귀에 빠뜨릴 위험도 존재한다.
            
            ```java
            public class SmellPoint extends Point {
                private String smell;
            
                public SmellPoint(int x, int y, String smell) {
                    super(x, y);
                    this.smell = smell;
                }
            
            		@Override public boolean equals(Object o) {
                    if (!(o instanceof Point)) {
                        return false;
                    }
            
                    if (!(o instanceof SmellPoint)) {
                        return o.equals(this);
                    }
            
            	      return super.equals(o) && ((SmellPoint) o).smell == smell;
            		}
            }
            ```
            
            ```java
            ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
            SmellPoint p2 = new SmellPoint(1, 2, "smell");
            
            p2.equals(p1);
            ```
            
- 그럼 해법은 무엇인가?
    - 사실, 이 현상은 모든 객체 지향 언어의 동치관계에서 나타나는 근본적인 문제이다.
    - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
        - 이 말은, instanceof 검사를 getClass 검사로 바꾸면 해결될 것이라고 들릴 수도 있다.
        - 아래의 예제를 살펴보자.

### 세번째 예제(리스코프 치환 원칙 위반)

- [리스코프 치환 원칙이란?](https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-LSP-%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84-%EC%B9%98%ED%99%98-%EC%9B%90%EC%B9%99)
    - 서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다는 것.
    - 교체할 수 있다는 말은, 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행이 보장되어야 한다는 의미.

```java
@Override public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass()) {
        return false;
    }

    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

```java
// 주어진 점이 반지름이 1인 원 안에 있는지 판별하는 메서드

private static final Set<Point> unitCircle = Set.of(
        new Point(1, 0), new Point(0, 1),
      new Point(-1, 0), new Point(0, -1));

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```

```java
// 만들어진 인스턴스의 개수를 생성자에서 세보기 위함

public class CounterPoint extends Point {

    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}
```

- CounterPoint 인스턴스를 onUnitCircle 메서드에 넘기면?
    
    → Point 클래스가 아니므로, false 반환
    
- 리스코프 치환 원칙을 위반한다. (자식 클래스가 부모 클래스로 대체될 수 없음)
- 위에서 볼 수 있듯, 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없다.
- 실제로 자바 라이브러리에도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있다.
    
    ex) `java.sql.Timestamp`
    
    ```java
    /**
     * ...
     * As a result, the {@code Timestamp.equals(Object)}
     * method is not symmetric with respect to the
     * {@code java.util.Date.equals(Object)}
     * method.
     * ...
     */
    public class Timestamp extends java.util.Date {
    
        private int nanos;
        
    }
    ```
    
    - equals 메서드는 대칭성을 위반하고 있다.
    - 따라서, API 설명에 Date와 섞어서 사용할 때의 주의사항을 언급하고 있다.
- 괜찮은 우회 방법이 있다!

## 상속 대신 컴포지션을 사용하라

- 컴포지션?
    - 기존 클래스가 새로운 클래스의 구성 요소로 쓰이는 것.

```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        this.point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

# 양질의 equals 메서드 구현 방법

### 바람직한 null 체크 방법

- 대부분의 코드에서 다음과 같이 null을 체크하곤 한다.
    
    ```java
    @Override public boolean equals(Object o) {
        if (o == null) {
            return false;
        }
    }
    ```
    
    - 하지만, 이러한 검사는 필요하지 않다.
        - instanceof에서 첫 번째 피연산자가 null이면 false를 반환하기 때문.
        - 따라서 다음과 같이 작성하는 것이 낫다.
            
            ```java
            @Override public boolean equals(Object o) {
                if (!(o instanceof MyType)) {
                    return false;
                }
            }
            ```
            

## 양질의 equals 메서드 구현 단계

1. == 연산자를 사용해서 입력이 자기 자신의 참조인지 확인한다.
    - 단순 성능 최적화용
    - 동등 비교 방법
        
        
        | 타입 | 방법 |
        | --- | --- |
        | 기본 타입 (float, double 제외) | == |
        | float | Float.compare(float, float) |
        | double | double.compare(double, double) |
        | 참조 타입 | 각각의 equals 메서드 |
    - 어떤 필드를 먼저 비교?
        - 다를 가능성이 더 크거나
        - 비교하는 비용이 싼
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 그렇지 않다면 false 반환
3. 입력을 올바른 타입으로 형변환한다.
    - instanceof 검사를 했기 때문에 100% 성공하는 작업
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’필드들이 모두 일치하는지 하나씩 검사한다.
    - 하나라도 다르면 false 반환

```java
// 순서대로 적용해보면 다음과 같은 형태가 될 것이다.

public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        // 1. == 연산자를 사용해서 입력이 자기 자신의 참조인지 확인한다.
        if (o.equals(this)) {
            return true;
        }

        // 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
        if (!(o instanceof Point)) {
            return false;
        }

        // 3. 입력을 올바른 타입으로 형변환한다.
        Point p = (Point) o;

        // 4. 입력 객체와 자기 자신의 대응되는 ‘핵심’필드들이 모두 일치하는지 하나씩 검사한다.
        return p.x == x && p.y == y;
    }
}
```

- 이후, 일반 규약을 만족했는지 테스트할 것 → (AutoValue 프레임워크를 활용할 것.)
    - 대칭적인지?
    - 추이성이 있는지?
    - 일관적인지?

# 주의사항

- equals를 재정의할 땐 hashCode도 반드시 재정의 ⭕️
- 너무 복잡하게 해결 ❌
    - ex) 파일의 심볼릭 링크
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언 ❌
    - 오버라이딩이 아닌, 오버로딩에 해당한다.

# 핵심

- 꼭 필요한 경우가 아니면 equals를 재정의하지 말자.
- 재정의를 해야한다면
    1. 그 클래스의 핵심 필드 모두를 빠짐없이 비교한다.
    2. 다섯 가지 규약을 지키는지 확인한다.
