# Item 81. wait와 notify보다는 동시성 유틸리티를 애용하라

## wait, notify 대신 고수준 동시성 유틸리티를 사용하자

이제는 `wait`와 `notify`를 사용해야 할 이유가 많이 줄었다.

**`wait`와 `notify`는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자**.

`java.util.concurrent`의 고수준 유틸리티의 범주는 **동시성 컬렉션**, **실행자 프레임워크**, **동기화 장치**로 나눌 수 있다.

### 동시성 컬렉션
---

**동시성 컬렉션**은 `List`, `Queue`, `Map` 같은 **표준 컬렉션 인터페이스에 동시성을 추가해 구현한 고성능 컬렉션**이다. 높은 동시성을 위해 동기화를 각자의 내부에서 수행한다. **동시성 컬렉션에서 동시성을 무력화하는 건 불가능하고, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다**.

#### 상태 의존적 수정 메서드

동시성 컬렉션은 **여러 메서드를 원자적으로 묶어 호출하는 것도 불가능**하다. **여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드가 추가**되었다. 이 메서드들은 매우 유용하여 자바 8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드 형태로 추가되었다.

``` java
// ConcurrentMap으로 구현한 동시성 정규화 맵
public class Intern {
    // 코드 81-1 ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다. (432쪽)
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

   public static String intern(String s) {
        // 주어진 키에 매핑된 값이 있으면 해당 값 반환, 없으면 추가하고 null 반환
       String previousValue = map.putIfAbsent(s, s);

       return previousValue == null ? s : previousValue;
   }
}
```

``` java
// ConcurrentMap으로 구현한 동시성 정규화 맵
public class Intern {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

    // 코드 81-2 ConcurrentMap으로 구현한 동시성 정규화 맵 - 더 빠르다! (432쪽)
    public static String intern(String s) {
        String result = map.get(s); // ConcurrentMap은 get 같은 검색 기능에 최적화되어 있어 빠름

        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result == null)
                result = s;
        }
        return result;
    }
}
```

저자는 위 코드가 `String.intern`보다 6배나 빠르다고 하지만, 내가 하나의 문자열에 대해서 단지 메서드 실행만 했을 때는 `String.intern`이 더 빨랐다. 저자는 어떻게 테스트한 건지 궁금하다...

**동시성 컬렉션은 동기화한 컬렉션의 성능을 뛰어 넘었다**. **`Collections.synchronizedMap`보다는 `ConcurrentHashMap`을 사용하는 게 훨씬 좋다**. 동시성 맵으로 교체하는 것만으로 **동시성 애플리케이션의 성능이 극적으로 개선**된다.

### 실행자 프레임워크
---

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.

`Queue`를 확장한 `BlockingQueue`에 추가된 메서드 중 `take`는 큐의 첫 원소를 꺼내고, 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다. 이러한 특성으로 `ThreadPoolExecutor`를 포함한 **대부분의 실행자 서비스 구현체에서 `BlockingQueue`를 사용**한다.

### 동기화 장치
---

**동기화 장치**는 **스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율**할 수 있게 해준다. 가장 자주 쓰이는 동기화 장치는 `CountDownLatch`와 `Semaphore`다. 가장 강력한 동기화 장치는 `Phaser`다. `CyclicBarrier`와 `Exchanger`는 덜 쓰인다.

#### CountDownLatch
---

`CountDownLatch`는 일회성 장벽으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. 이 래치의 생성자는 `int` 값을 유일하게 받는데, 이 값이 `countDown` 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.

``` java
// 코드 81-3 동시 실행 시간을 재는 간단한 프레임워크 (433-434쪽)
public class ConcurrentTimer {
    private ConcurrentTimer() { } // 인스턴스 생성 불가

    public static long time(
        Executor executor,  // concurrency 만큼 스레드를 생성할 수 있어야 함 (그러지 않으면 스레드 기아 교착상태에 빠짐)
        int concurrency,    // 스레드 개수
        Runnable action) throws InterruptedException {

        CountDownLatch ready = new CountDownLatch(concurrency); // 작업자 스레드들의 준비 상태
        CountDownLatch start = new CountDownLatch(1);   // 작업자 스레드들의 시작 상태
        CountDownLatch done  = new CountDownLatch(concurrency); // 작업자 스레드들의 작업 완료 상태

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                try {
                    start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    action.run();
                } catch (InterruptedException e) {
                    // 스레드의 run()을 정상 종료시키기 위함
                    Thread.currentThread().interrupt(); 
                } finally {
                    done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                }
            });
        }

        ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
        long startNanos = System.nanoTime();    // 정확한 측정을 위해 System.nanoTime() 사용 (System.currentTimeMillis는 지양)
        start.countDown(); // 모든 준비된 작업자들을 깨운다.
        done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
        return System.nanoTime() - startNanos;
    }
}
```

위와 같은 기능을 `wait`와 `notify`만으로 구현하려면 아주 난해하고 지저분한 코드가 탄생하지만, `CountDownLatch`를 사용하면 직관적으로 구현할 수 있게 된다.

<br>

## 어쩔 수 없이 wait, notify를 사용해야 할 때

새로운 코드라면 언제나 `wait`와 `notify`가 아닌 동시성 유틸리티를 사용해야 한다. 그러나 어쩔 수 없이 다뤄야 할 때도 있을 것이다.

### wait 사용법
---

**`wait` 메서드**는 **스레드가 어떤 조건이 충족되기를 기다리게 할 때** 사용한다. **락 객체의 `wait` 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다**.

#### wait 메서드를 사용하는 표준 방식

``` java
synchronized (obj) {
    while (<조건이 충족되지 않았다>) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

**`wait` 메서드를 사용할 땐느 반드시 대기 반복문 코드를 사용하자. 반복문 밖에서는 절대로 호출하지 말자.**

**조건이 이미 충족되었다면, `wait`을 건너띄게 한 것은 응답 불가 상태를 예방하는 조치**다. 만약 조건이 이미 충족되었는데 스레드가 `notify(또는 notifyAll)` 메서드를 먼저 호출한 후 대기 상태로 빠진다면, 그 스레드는 다시 깨울 수 있다고 보장할 수 없다. 스레드가 `wait` 되기 전에 `notify`로 깨워지면, 이후에 다시 `notify` 메서드가 실행되지 않는 이상 계속 대기하게 된다.

### 조건이 만족하지 않아도 스레드가 깰 수 있는 상황
---

- **스레드가 `notify`를 호출한 다음 대기 중이던 스레드가 깨어나느 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다**.

- **조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 `notify`를 호출**한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 `wait`는 모두 이 문제에 영향을 받는다.

- 깨우는 스레드는 지나치게 관대해서, **대기 중인 스레드 중 일부만 조건이 충족되어도 `notifyAll`을 호출해 모든 스레드를 깨울 수도 있다**.

- **대기 중인 스레드가 드물게 `notify` 없이도 깨어나는 경우**가 있다. 이는 **허위 각성**(Spurious Wakeup)이라고는 현상이다.

### notify보단 notifyAll을 사용하자
---

`notify`는 하나의 스레드만 깨우고, `notifyAll`은 모든 스레드를 깨운다. 둘 중 어떤 메서드를 선택해야 할지 고민할 수 있다.

**일반적으로 언제나 `notifyAll`을 사용하는 게 합리적이고 안전한 조언**이 될 것이다. 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다. **조건이 충족하지 않는 다른 스레드들도 깨어날 수 있지만**, 영향을 주지 않을 것이다. 깨어난 스레드들은 **기다리던 조건이 충족되었느지 확인하여, 충족되지 않았다면 다시 대기할 것**이기 때문이다.

**모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 `notify`를 사용해 최적화**를 할 수 있다. 그러나 이 전제조건이 만족하더라도 `notify` 대신 `notifyAll`을 사용해야 하는 이유가 있다. 

외부로 공개된 객체에 대해 실수로 또는 악의적으로 `notify`를 호출하는 상황에 대비하기 위해 `wait`를 반복문 안에서 호출했듯이, `notify` 대신 `notifyAll`을 사용하면 관련 없는 스레드가 실수로 또는 악의적으로 `wait`를 호출하는 공격으로부터 보호할 수 있다. 만약 그런 스레드가 `notify`를 삼켜버린다면 꼭 깨어났어야 할 스레드들이 영원히 대기하게 될 수 있다.

<br>

## 정리

- 코드를 새로 작성한다면 `wait`과 `notify`를 쓸 이유가 거의 없다.

- `wait`는 항상 표준 관용구에 따라 `while` 문 안에서 호출하자.

- 일반적으로 `notify`보다는 `notifyAll`을 사용해야 한다.

- `notify`를 사용한다면 응답 불가 상태에 빠지지 않도록 주의하자!
