## 배열이나 리스트에서 원소를 꺼낼 때 ordianl 메서드로 인덱스를 얻는 코드

```java
class Plant {  
    final String name;  
    final LifeCycle lifeCycle;  
  
    Plant(String name, LifeCycle lifeCycle) {  
        this.name = name;  
        this.lifeCycle = lifeCycle;  
    }  
    @Override  
    public String toString() {  
        return name;  
    }
    
	enum LifeCycle {PERENNIAL, BIENNIAL, ANNUAL}
}
Plant[] garden = {  
        new Plant("바질", LifeCycle.ANNUAL),  
        new Plant("캐러웨이", LifeCycle.BIENNIAL),  
        new Plant("딜", LifeCycle.ANNUAL),  
        new Plant("라벤더", LifeCycle.PERENNIAL),  
        new Plant("파슬리", LifeCycle.BIENNIAL),  
        new Plant("로즈마리", LifeCycle.PERENNIAL)  
};
```

```java
// 코드 37-1 ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것! (226쪽)  
Set<Plant>[] plantsByLifeCycleArr =  
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];  
for (int i = 0; i < plantsByLifeCycleArr.length; i++)  
    plantsByLifeCycleArr[i] = new HashSet<>();  
for (Plant p : garden)  
    plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);  
// 결과 출력  
for (int i = 0; i < plantsByLifeCycleArr.length; i++) {  
    System.out.printf("%s: %s%n",  
            Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);  
}
// 동작 O
```
그러나 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않는다.

배열은 각 인덱스의 의미를 모르니 직접적으로 인덱스에 대한 정보를 제공해야 한다.  
`LifeCycle.values()[i]`를 사용하여 각 배열 인덱스의 의미(ex: PERENNIAL, BIENNIAL, ANNUAL)를 직접 명시

여기서 배열은 열거 타입 상수를 값으로 매핑하는 일을 하는데 `Map`을 사용할 수도 있지만, 열거 타입을 키로 사용하도록 설계한 아주 빠른 `Map`구현체가 존재.

## EnumMap 사용
```java
// 코드 37-2 EnumMap을 사용해 데이터와 열거 타입을 매핑한다. (227쪽)  
Map<LifeCycle, Set<Plant>> plantsByLifeCycle =  
        new EnumMap<>(Plant.LifeCycle.class);  
for (Plant.LifeCycle lc : Plant.LifeCycle.values())  
    plantsByLifeCycle.put(lc, new HashSet<>());  
for (Plant p : garden)  
    plantsByLifeCycle.get(p.lifeCycle).add(p);  
System.out.println(plantsByLifeCycle);
```

- 더 짧고 명료
- 안전하지 않은 형변환 ❌
- Key가 문자열이니 직접 명시할 필요 ❌
- 배열 인덱스를 쓰지 않으니 `ArrayIndexOutOfBoundsException` 걱정 ❌

내부 구현 방식을 안으로 숨겨서 `Map`의 타입 안전성과 배열의 성능을 모두 얻어냄.

`EnumMap`의 생성자가 받는 키 타입의 `Class`객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보 제공.
```java
public EnumMap(Class<K> keyType) {  
    this.keyType = keyType;  
    keyUniverse = getKeyUniverse(keyType);  
    vals = new Object[keyUniverse.length];  
}
```

스트림을 사용하면 코드를 더 줄일 수가 있다.

```java
// 코드 37-3 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다! (228쪽)  
System.out.println(Arrays.stream(garden)  
        .collect(groupingBy(p -> p.lifeCycle)));  
  
// 코드 37-4 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다. (228쪽)  
System.out.println(Arrays.stream(garden)  
        .collect(groupingBy(p -> p.lifeCycle,  
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```

첫번째 코드는 `EnumMap`이 아닌 고유한 맵 구현체를 사용하여 `EnumMap`의 이점이 사라진다.

**`Map` 구현체가 왜 `EnumMap`보다 공간과 성능 이점이 떨어지는가?**  
- `EnumMap`은 배열 기반으로 동작하므로 성능이 더 뛰어남.
- 열거형의 상수 개수가 고정되어 있거나 작은 경우, `EnumMap`은 메모리와 처리 속도 면에서 유리.
- 맵의 키로 열거형이 아닌 다른 타입이 들어갈 가능성이 생김. 즉, 타입 안전성이 떨어질 수 있음.

스트림을 사용하면 `EnumMap`만 사용했을 때와는 다르게 동작.
```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle =  
        new EnumMap<>(Plant.LifeCycle.class);  
for (Plant.LifeCycle lc : Plant.LifeCycle.values())  
    plantsByLifeCycle.put(lc, new HashSet<>());  
for (Plant p : garden)  
    plantsByLifeCycle.get(p.lifeCycle).add(p);  
System.out.println(plantsByLifeCycle); // {PERENNIAL=[], BIENNIAL=[파슬리, 캐러웨이], ANNUAL=[바질, 딜]}

System.out.println(Arrays.stream(garden)  
        .collect(groupingBy(p -> p.lifeCycle,  
                () -> new EnumMap<>(LifeCycle.class), toSet())));
// {BIENNIAL=[파슬리, 캐러웨이], ANNUAL=[바질, 딜]}
```


## ordinal을 2번 쓴 배열
```java
// 코드 37-5 배열들의 배열의 인덱스에 ordinal() 사용 - 따라 하지 말것!
public enum Phase {  
    SOLID, LIQUID, GAS;  
    public enum Transition {  
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;  

		// 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
		private static final Transition[][] TRANSITIONS = {
			{ null, MELT, SUBLIME },
			{ FREEZE, null, BOIL },
			{ DEPOSIT, CONDENSE, null }
		}
  
        public static Transition from(Phase from, Phase to) {  
			return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}

TRANSITIONS[1][2] // 무엇을 의미하는지 알 수 없음
```

열거형에 새로운 상수를 추가하거나 순서를 변경하면, 배열의 크기와 매핑이 깨질 위험이 있다.

## 중첩 EnumMap
```java
// 코드 37-6 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다. (229-231쪽)  
public enum Phase {  
    SOLID, LIQUID, GAS;  
    public enum Transition {  
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),  
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),  
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);  
  
        private final Phase from;  
        private final Phase to;  
        Transition(Phase from, Phase to) {  
            this.from = from;  
            this.to = to;  
        }  
  
        // 상전이 맵을 초기화한다.  
        private static final Map<Phase, Map<Phase, Transition>>  
                m = Stream.of(values())
                .collect(groupingBy(t -> t.from, 
                () -> new EnumMap<>(Phase.class),  
                toMap(t -> t.to, t -> t,  
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));  
  
        public static Transition from(Phase from, Phase to) {  
            return m.get(from).get(to);  
        }  
    }
}
```
상전이 맵을 초기화하는 코드는 복잡함.

`Map<Phase, Map<Phase, Transition>>`은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응 시키는 맵" 이라는 뜻. 말이 어렵다...

```java
Stream.of(values())
    .collect(groupingBy(
        t -> t.from,                     // 첫 번째 Map의 키: Transition의 `from` 상태
        () -> new EnumMap<>(Phase.class), // 첫 번째 Map의 자료구조: EnumMap
        toMap(
            t -> t.to,                   // 두 번째 Map의 키: Transition의 `to` 상태
            t -> t,                      // 두 번째 Map의 값: Transition 객체
            (x, y) -> y,                 // 병합 함수 (중복이 발생하면 마지막 값을 사용)
            () -> new EnumMap<>(Phase.class) // 두 번째 Map의 자료구조: EnumMap
        )
    ));
```
GPT 한테 부탁했습니다

**만약 여기에 새로운 상태인 PLASMA 를 추가한다면**
- ordinal 2번 쓴 배열에선 
	- 새로운 상수를 `Phase`에 1개 `Phase.Transition`에 2개 추가
	- 원소 9개짜리인 배열들의 배열을 원소 16개짜리로 교체

```java
public enum Phase {  
    SOLID, LIQUID, GAS, PLASMA;  
    public enum Transition {  
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT, IONIZE, DEIONIZE;  

		// 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
        private static final Transition[][] TRANSITIONS = {
            { null, MELT, SUBLIME, null }, 
            { FREEZE, null, BOIL, null },
            { DEPOSIT, CONDENSE, null, IONIZE },
            { null, null, DEIONIZE, null }

        };
  
        public static Transition from(Phase from, Phase to) {  
			return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```
번거롭고 문제가 생길 여지가 많다.

```java
SOLID, LIQUID, GAS, PLASMA;  
public enum Transition {  
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),  
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),  
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),  
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
```
중첩 `EnumMap`에선 상태 목록에 PLASMA, 전이 목록에 IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS)만 추가하면 끝.

## 결론
배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 좋지 않으니, `EnumMap`을 사용하라.
다차원 관계는 중첩`EnumMap`으로 표현하라.

