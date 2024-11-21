# equals를 재정의하면 hashCode를 재정의 해야하는 이유

### Object 명세 일부

> “equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.”
> 

### 즉, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

이러한 규칙을 어길 경우 해시함수를 통해 반환된 해시코드 값을 이용하는 Collections에서 문제를 일으킬 것이다.

- Collections와 hashCode의 관계
    
    HashMap으로 생각해보자.
    
    - map.put(key, value)
        1. key 객체의 hashCode() 메서드를 호출하여 hash 값을 얻는다.
        2. hash값을 통해 버킷에 저장한다.
    - map.get(key, value)
        1. key 객체의 hashCode() 메서드를 호출하여 hash 값을 얻는다.
        2. hash값을 통해 버킷에 존재하는 데이터를 읽어온다.
    

예시를 통해 알아보자.

```java
public class Point {

    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (o.equals(this)) {
            return true;
        }

        if (!(o instanceof Point)) {
            return false;
        }

        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

```java
Map<Point, String> m = new HashMap<>();
m.put(new Point(1, 2), "좌표1");
System.out.println(m.get(new Point(1, 2)));
// 출력: null
```

- equals 메서드를 재정의하여 서로 논리적으로 동치임에도 hashCode를 재정의하지 않았기에, null이 출력된다.

# hashCode 작성법

## 최악의 구현

```java
@Override public int hashCode() { return 42; }
```

- 모든 객체에게 똑같은 hash 값을 내어주므로, 해시테이블은 연결 리스트처럼 동작하게 된다.
- 그럼, hash의 O(1)이라는 이점을 O(n)으로 떨어뜨리게 된다.

## 좋은 HashCode 작성 방법

1. int 변수 result를 선언한 후 값 c로 초기화한다.
    - 이때 c는 해당 객체의 첫 번째 핵심 필드를 단계 2.a 방식으로 계산한 해시코드이다.
        - 여기서 핵심 필드란 equals 비교에 사용되는 필드를 말한다.
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c를 계산한다.
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
        2. 참조 타입 필드면서, 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다. 
        계산이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다.
        필드의 값이 null이면 0을 사용한다.
            - [표준형에 대한 설명이 필요하다면 참고하기 좋은 블로그](https://blog.naver.com/kbh3983/221000277434)
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 
        이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b 방식으로 갱신한다. 
        배열에 핵심 원소가 하나도 없다면 단순히 상수를 사용한다. 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다. 코드로는 다음과 같다.
        
        ```java
        result = 31 * result + c;
        ```
        
3. result를 반환한다.

```java
// 예시

@Override
public int hashCode() {
    int result = Integer.hashCode(x);
    result = 31 * result + Integer.hashCode(y);
    return result;
}
```

## Objects.hash()

Objects 클래스의 hash() 메서드

⇒ 임의의 개수만큼 객체를 받아, 해시코드를 계산해주는 메서드

- 장점
    - 구현 쉬움
        - 앞서 요령대로 구현한 코드와 비슷한 수준의 hashCode 함수를 단 한 줄로 작성할 수 있음.
- 단점
    - 느린 속도
        - 입력 인수를 담기 위한 배열 생성 + 기본 타입의 경우 박싱과 언박싱
- 그럼 언제 사용?
    - 성능에 민감하지 않은 상황에서만.

### 주의사항

- equals 비교에 사용되지 않은 필드는 반드시 제외해야 한다.
    - 그렇지 않으면 hashCode의 규약을 어기게 될 여지가 있음.
- 해시 키로 사용되는 클래스가 불변이고, 해시코드를 계산하는 비용이 크다면, 캐싱을 고려한다.
    - 매번 계산하기 보다는, 인스턴스가 만들어질 때 해시코드를 미리 계산해 두는 방식.
- 해시 키로 사용되지 않는 경우라면, 지역 초기화(lazy initialization) 전략을 고려한다.
- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안 된다.
    
    → 해시테이블의 성능을 떨어뜨리기 때문
    
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말라.
    
    → 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.
