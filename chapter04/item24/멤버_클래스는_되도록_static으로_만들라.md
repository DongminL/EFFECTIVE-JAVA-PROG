## 중첩 클래스
다른 클래스 안에 정의된 클래스

중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

### 중첩 클래스의 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스
이 중 첫번째를 제외한 나머지는 내부 클래스(Inner Class)에 해당.

#### 정적 멤버 클래스 & 비정적 멤버 클래스
```java
public class Outer {  
  
    public class Inner {  
    }  
    public static class StaticInner {  
    }
}

////

Outer.Inner inner = new Outer().new Inner();  
Outer.StaticInner staticInner = new Outer.StaticInner();
```
정적 멤버 클래스는
- 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 같음.
- 정적 멤버 클래스는 다른 정적 멤버와 똑같은 규칙을 적용받음(private 선언 시 바깥 클래스에서만 접근 가능)
- 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 함.

비정적 멤버 클래스는
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결됨.
- 비정적 멤버 클래스는 바깥 클래스의 인스턴스에서 만들 수 있음.
	- 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸림
- 비정적 멤버 클래스는 외부 클래스의 참조를 유지하므로, 외부 클래스 인스턴스가 메모리에서 해제되지 못하는 경우가 발생.
- 어댑터를 정의할 때 자주 쓰임.
	- 컬렉션 뷰 구현 시 반복자(iterator)나 뷰 객체가 외부 클래스의 상태를 직접 참조할 수 있어 구현이 간결.

**멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만드는 것이 좋다.**

##### private 정적 멤버 클래스
```java
public class OuterClass {
    private static class Helper {
        static int calculateSquare(int x) {
            return x * x;
        }
    }

    public static int getSquare(int x) {
        return Helper.calculateSquare(x);
    }
}

public static void main(String[] args) {
	System.out.println(OuterClass.getSquare(5)); // 출력: 25
}
```
중첩 클래스에서 static을 빼도 동작은 하지만 외부 참조가 생겨 공간과 시간을 낭비하게 된다.

**멤버 클래스가 공개된 public이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요.**
	-> 향후 릴리스에서 하위호환성을 깨지 않기 위해

#### 익명 클래스
익명 클래스는 바깥 클래스의 멤버가 아니다.  
멤버와 달리 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.

그리고 오직 비정적인 문맥에서 사용될 때만 바깥 클래스의 인스턴스를 참조할 수 있다

```java
public class OuterClass {
    private static int staticField = 10; // 정적 필드
    private int instanceField = 20;     // 비정적(인스턴스) 필드
    //....
}
```
정적 문맥에서는 익명 클래스가 **바깥 클래스의 인스턴스를 참조하지 못하므로 비정적 멤버에 접근 불가**합니다.

익명 클래스의 또 다른 주 쓰임은 정적 팩토리 메서드를 구현할 때이다.
```java
// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
```

### 지역 클래스
네 가지 중첩 클래스 중 가장 드물게 사용.

- 지역변수를 선언할 수 있는 곳이면 어디서든 선언이 가능.  
- 유효 범위도 지역변수와 같다.
- 다른 세 중첩 클래스와의 공통점도 하나씩 가지고 있다.
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 가독성을 위해 짧게 작성.
- 정적 멤버는 가질 수 없다.

정적 멤버를 가질 수 없다 부분에서 신기해서 직접 코드를 쳐보니
```java
public class Outer {  
  
    public class Inner {  
    }  
    public static class StaticInner {  
    }  
    private class Helper {  
        static int calculateSquare(int x) {  
            return x * x;  
        }  
    }  
  
    public static int getSquare(int x) {  
        return Helper.calculateSquare(x);  
    }  
  
    public void method() {  
        class Local {  
            static int local = 10;  
        }  
        System.out.println(Local.local);  
    }  
  
    public static void main(String[] args) {  
        new Outer().method();  
    }  
}
```
출력 값이 10이라고 잘 나옵니다..... 컴파일 에러도 없고 런타임 에러도 없고

JAVA 8 기준  
JLS 8.1.3 에서는  지역 클래스에선 `static final` 상수는 가능하지만 `static`은 컴파일 에러가 난다고 써져있습니다.  
[JLS 8.1.3 문서](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.1.3)

JDK 버전이 올라가면서 컴파일 에러가 나지 않고 잘 동작을 하는거 같은데 그래도 명세에 지켜서 지역 클래스에서 정적 멤버를 쓰지 않는게 좋을거 같다고 생각합니다!
(아니면 final을 자동으로 붙여주는 기능이 추가?)

## 결론
1. 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만들기.
2. 멤버 클래스 인스턴스가 각각이 바깥 인스턴스 참조한다면 비정적, 아니라면 정적으로 만들기.
3. 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스.
4. 아니라면 지역 클래스
