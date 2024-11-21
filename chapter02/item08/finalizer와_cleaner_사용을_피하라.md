## 두 가지 객체 소멸자
### finalizer
> finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
> finalizer는 나름의 쓰임새가 몇 가지 있긴 하지만 기본적으로 **'쓰지말아야'** 한다

finalizer는 객체가 GC에 의해 회수되기 전에 호출되는 메서드.
Object 클래스의 finalize() 메서드를 재정의하여 사용.

```java
public class FinalizerExample {  
  
    // 생성자에서 리소스 할당  
    public FinalizerExample() {  
        System.out.println("객체 생성!");  
    }  
  
    // finalize() 메서드 재정의  
    @Override  
    protected void finalize() throws Throwable {  
        try {  
            // 객체가 GC에 의해 회수되기 전 호출됨  
            System.out.println("Finalizer 실행: 리소스 정리 중...");  
        } finally {  
            // 부모 클래스의 finalize 호출  
            super.finalize();  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        FinalizerExample example = new FinalizerExample();  
  
        // 객체를 null로 설정하여 더 이상 참조되지 않도록 설정  
        example = null;  
  
        // 강제로 GC 실행 요청  
        System.gc();  
        Thread.sleep(100); // GC 수행 시간 대기  
  
        System.out.println("메인 메서드 종료");  
    }  
}
/* 출력
객체 생성!
Finalizer 실행: 리소스 정리 중...
메인 메서드 종료
*/
```

#### 특징
- 자동 호출: JVM이 GC를 수행할 때 필요하다고 판단되면 호출
- 사용 시점 예측 불가: 정확한 호출 시점 예측이 어려움
- 성능: GC 작업을 지연시킬 수 있으며, 성능에 악영향을 줄 수 있음
- 리소스 누수 위험: finalize()의 실행이 실패하면 자원이 제대로 해제되지 않을 수 있음

Java9 부터는 deprecated API로 지정됐고 Java18 이후로는 퇴역될 예정이라 합니다....
[finalize 퇴역](https://www.itworld.co.kr/news/224419)

### cleaner
Cleaner는 Java 9에서 도입된 클래스

```java
public class CleanerExample {  
  
    // Cleaner 인스턴스 생성  
    private static final Cleaner cleaner = Cleaner.create();  
  
    // 정리 작업을 정의하는 Runnable 구현체  
    static class State implements Runnable {  
        private String resourceName;  
  
        public State(String resourceName) {  
            this.resourceName = resourceName;  
        }  
  
        @Override  
        public void run() {  
            // 정리 작업 수행  
            System.out.println(resourceName + " 리소스 정리 중!");  
        }  
    }  
  
    // 실제 객체와 정리 작업을 연결  
    private final Cleaner.Cleanable cleanable;  
  
    public CleanerExample(String resourceName) {  
        // 객체와 정리 작업을 등록  
        State state = new State(resourceName);  
        cleanable = cleaner.register(this, state);  
    }  
  
  
    public static void main(String[] args) {  
        CleanerExample example = new CleanerExample("리소스1");  
  
        // 객체를 더 이상 참조하지 않음 -> example 이 GC 대상이 됨  
        example = null;  
  
        // 강제로 GC 실행 요청  
        System.gc();  
  
        // GC 실행 시간을 보장하기 위해 대기  
        try {  
            Thread.sleep(1000); // 1초 대기  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        System.out.println("메인 메서드 종료");  
    }  
}
```
#### 특징
- 명시적 실행: GC와 독립적으로 동작하며, Cleaner 객체는 백그라운드 스레드를 사용하여 정리 작업 수행
- 성능: GC 지연 최소화

Cleaner는 Finalizer보다는 덜 위험하지만, 여전히 예측 불가능하고, 느리고 일반적으로 불필요하다.

## 대안
```AutoCloseable```을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.

### cleaner와 finalizer의 쓰임새?
1. 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할.

	즉시 호출되리라는 보장은 없지만, 언젠간 자원 회수를 해주기 때문
	
	`FileInputStream`, `FileOutputStream'`, `ThreadPoolExecutor` 안전망 역할의 finalizer를 제공한다고 책에 쓰여있지만
	
  <img width="472" alt="image" src="https://github.com/user-attachments/assets/175368c9-26b7-4b51-b395-647c4430c057">

	
	finalize는 신뢰할 수 없으며 사용을 권장하지 않는다고 API 문서에 쓰여있다.

2. 네이티브 피어 리소스 관리
	네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 객체를 말합니다. **자바 객체와 네이티브 코드 사이의 다리 역할**  
	
	성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close 메서드를 사용


### 까다로운 Cleaner 사용법
```java
public class Room implements AutoCloseable {  
    private static final Cleaner cleaner = Cleaner.create();  
    private final State state; // 방의 상태 cleanable 과 공유  
    private final Cleaner.Cleanable cleanable; // cleanable 객체. 수거 대상이 되면 방 청소  
  
    public Room(int numJunkPiles) {  
        this.state = new State(numJunkPiles);  
        this.cleanable = cleaner.register(this, state);  
    }  
  
    @Override  
    public void close() throws Exception {  
        cleanable.clean();  
    }  
  
    // 청소가 필요한 자원  
    private static final class State implements Runnable {
    // State는 절대로 Room 인스턴스 참조 X  
        int numJunkPiles;  
        /*  
		 현실적으로 만들려면 이 필드는 네이티브 피어를 가리키는 
		 포인터를 다음 final long 변수여야 한다. 
		 */
  
        public State(int numJunkPiles) {  
            this.numJunkPiles = numJunkPiles;  
        }  
  
        // close 메서드나 cleaner가 호출  
        @Override  
        public void run() {  
            System.out.println("방 청소");  
            numJunkPiles = 0;  
        }  
    }  
}

/////

public static void main(String[] args) {  
    try(Room room = new Room(7)) {  
        System.out.println("HI");  
    } catch (Exception e) {  
        throw new RuntimeException(e);  
    }  
  
    new Room(99);  
    System.out.println("HELLO");  
}
/* 출력
HI
방 청소
HELLO
*/
```

State 클래스를 정적 클래스로 만든 이유
1. Room 인스턴스를 참조하는 경우 순환참조가 생겨 GC가 Room 인스턴스를 회수해가지 못하기 때문.
2. 정적이 아닌 내부 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문





