@Override 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

```java
// 코드 40-1 영어 알파벳 2개로 구성된 문자열(바이그램)을 표현하는 클래스 - 버그를 찾아보자. (246쪽)
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

		// 오버라이딩이 아닌, 오버로딩
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}

// 출력 결과: 260
```

- Object의 equals 메서드를 재정의하려면 매개변수 타입을 Object로 해야만 한다.
- 하지만, Bigram이라는 타입으로 지정했기에, 오버라이딩이 아닌 오버로딩이 된 것.
    - 따라서, 26이 아닌, 260이라는 결과가 출력된다.
- @Override 애너테이션을 사용하면, 컴파일 타임에 오류를 식별할 수 있다.

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof Bigram2))
        return false;
    Bigram2 b = (Bigram2) o;
    return b.first == first && b.second == second;
}
```

### 그러니, 상위 클래스의 메서드를 재정의하는 모든 메서드에 @Override 애너테이션을 달자.

- 예외는 한가지 뿐이다.
    - 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 @Override를 달지 않아도 된다.
        
        → 어차피 구체 클래스에 구현하지 않은 추상 메서드가 남아있다면 컴파일러가 알려주기 때문.
        

### @Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다.

- 인터페이스에 디폴트 메서드가 없음을 안다면, 이를 구현한 메서드에서는 @Override를 생략해 코드를 조금 더 깔끔히 유지해도 좋다.

### 정리

- 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달아 실수를 방지하자.
