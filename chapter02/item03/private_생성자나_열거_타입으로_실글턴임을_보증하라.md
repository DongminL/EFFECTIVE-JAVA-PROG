# 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

이 장에서는 자바에서 싱글턴을 만드는 방식을 설명합니다.

싱글턴 (singleton)

: 인스턴스를 오직 하나만 생성할 수 있는 클래스

일반적으로 싱글턴을 만드는 방식 두 가지

## 1. public static 멤버가 final 필드인 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	
	public void leaveTheBuilding() { ... }
}
```

### 특징

- private 생성자는 INSTANCE가 초기화될 때 딱 한 번만 호출된다.
- 외부에서 생성자를 호출할 방법이 없으므로, 전체 시스템에서 인스턴스가 하나임이 보장된다.

### 장점

- 해당 클래스가 싱글턴임이 명확히 드러난다.
- 간결하다.

## 2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }
	
	public void leaveTheBuilding() { ... }
}
```

### 특징

- 1번 방식의 특징이 그대로 적용된다.

### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.
    - 좀 더 풀어서 이야기하면, 싱글턴 인스턴스를 반환하던 것을 내부적으로 호출하는 스레드별로 다른 인스턴스를 넘겨주게 변경해도 API를 바꿀 필요는 없다는 의미.
- 제네릭 싱글턴 팩토리로 만들 수 있다. (아래는, GPT가 생산한 예시)
    
    ```java
    public class GenericSingletonSet {
        private static final Set<?> EMPTY_SET = Collections.emptySet();
    
        @SuppressWarnings("unchecked")
        public static <T> Set<T> emptySet() {
            return (Set<T>) EMPTY_SET;
        }
    }
    ```
    
    ```java
    Set<String> stringSet = GenericSingletonSet.emptySet();
    Set<Integer> integerSet = GenericSingletonSet.emptySet();
    
    // 동일 인스턴스 확인
    System.out.println(stringSet == integerSet); // true
    ```
    
- 정적 팩터리 메서드 참조를 공급자(supplier)로 사용할 수 있다.
    
    [Supplier의 Lazy Evaluation의 장점을 잘 보여준 블로그](https://m.blog.naver.com/zzang9ha/222087025042)
    

## 1, 2번 방식에서 싱글톤이 깨질 수 있는 경우

### 1. 리플렉션

```java
@Test
public void test()
    throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Elvis elvis1 = Elvis.INSTANCE;

    Class<Elvis> elvisClass = Elvis.class;
    Constructor<Elvis> constructor = elvisClass.getDeclaredConstructor();
    constructor.setAccessible(true); // private에 접근할 수 있도록 상태 변경
    Elvis elvis2 = constructor.newInstance();

    Assertions.assertThat(elvis1).isNotEqualTo(elvis2);
}

// true
// 즉, elvis1과 elvis2는 서로 다른 인스턴스.
```

- 이러한 공격을 방어하려면 두 번째 객체가 생성되려 할 때 예외를 던지도록 생성자를 수정해야 한다.
    
    ```java
    private Elvis() {
    	if(INSTANCE != null){
    		throw new RuntimeException("이미 인스턴스가 존재합니다.");
    	}
    }
    ```
    

### 2. 역직렬화

- 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 만들어진다.
    
    ```java
    @Test
    public void test() throws IOException, ClassNotFoundException {
        Elvis elvis1 = Elvis.INSTANCE;
    
        String fileName = "elvis.obj";
    
        // 직렬화
        ObjectOutputStream out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(fileName)));
        out.writeObject(elvis1);
        out.close();
    
        // 역직렬화
        ObjectInputStream in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(fileName)));
        Elvis elvis2 = (Elvis) in.readObject();
        in.close();
    
        Assertions.assertThat(elvis1).isNotEqualTo(elvis2);
    }
    
    // true
    // 즉, elvis1과 elvis2는 서로 다른 인스턴스.
    ```
    
    - 이를 방지하기 위해서는 모든 인스턴스 필드를 transient이라고 선언하고, readResolve 메소드를 제공해야 한다.
        
        ```java
        public class Elvis implements Serializable {
        
            public static final Elvis INSTANCE = new Elvis();
        
            private Elvis() {}
            public void leaveTheBuilding() {}
        
            private Object readResolve() {
                return INSTANCE;
            }   
        }
        ```
        
        ```java
        @Test
        public void test() throws IOException, ClassNotFoundException {
            Elvis elvis1 = Elvis.INSTANCE;
        
            String fileName = "elvis.obj";
        
            // 직렬화
            ObjectOutputStream out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(fileName)));
            out.writeObject(elvis1);
            out.close();
        
            // 역직렬화
            ObjectInputStream in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(fileName)));
            Elvis elvis2 = (Elvis) in.readObject();
            in.close();
        
            Assertions.assertThat(elvis1).isEqualTo(elvis2);
        }
        
        // true
        // 즉, elvis1과 elvis2는 서로 같은 인스턴스.
        ```
        

### 3. 원소가 하나인 열거 타입을 선언하는 방식

```java
public enum Elvis {
	INSTANCE;
	
	public void leaveTheBuilding() { ... }
}
```

- 저자가 가장 바람직하다고 주장하는 방식이다.
    
    > “조금 부자연스러워 보일 수는 있으나, 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.”
    > 
- public 필드 방식과 비슷하지만
    - 보다 간결하다.
    - 귀찮은 직렬화 과정을 거칠 필요가 없다.
    - 또한, 리플렉션 공격까지도 막아준다.
- 그럼 열거 타입으로 선언하는 것이 정답인가?
    - 싱글턴으로 구성할 클래스가 다른 클래스를 상속해야 하는 경우에는 사용할 수 없다.
        - enum은 클래스 상속이 불가능하므로
