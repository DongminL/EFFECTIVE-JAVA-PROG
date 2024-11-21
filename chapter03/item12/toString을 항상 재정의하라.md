# toString을 항상 재정의 해야하는 이유

```java
public class Object {
    /**
     * @apiNote
     * In general, the
     * {@code toString} method returns a string that
     * "textually represents" this object. The result should
     * be a concise but informative representation that is easy for a
     * person to read.
     * It is recommended that all subclasses override this method.
			 ...
     */
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
}
```

- toString의 일반 규약에 따르면 ‘간결하면서 사람이 읽기 쉬운 형태의 유익한 정보’를 반환해야 한다.
    - 즉, PhoneNumber@adbbd보다는 707-867-5309처럼 전화번호를 직접 알려주는 형태가 훨씬 유익하다.
- 또한, toString 규약은 “모든 하위 클래스에서 이 메서드를 재정의하라”고 한다.
    - 장점
        - 사용하기 편함
            - ex) map 객체 출력 시 `Jenny=PhoneNumber@adbbd`, `Jenny=707-867-5309` 무엇이 더 보기에 편한가?
        - 디버깅 용이

## 실전에서의 toString

그 객체가 가진 주요 정보를 모두 반환하는 것이 좋다.

### 하지만, 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면? 🤔

이상적으로는 스스로를 완벽히 설명하는 문자열(요약된 정보)을 제공한다.

ex) 전화번호부(총 149591개)

### 반환값의 포맷을 명시할 것인지 결정해야 한다

- 우선, 포맷 명시 여부와 관계없이 지켜야 할 사항
    - 의도를 명확히 밝힌다.
    - toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.
        - 그렇지 않으면, 반환값을 파싱해야 하는 소요가 생기고, 이는 낮은 성능과 생산으로 이어진다.
- 포맷을 명시하는 경우
    
    ```java
    /**
     * 이 전화번호의 문자열 표현을 반환한다.
     * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
     * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
     * 각각의 대문자는 10진수 숫자 하나를 나타낸다. 
     * ...
     */
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
    ```
    
    - 장점
        - 표준화 ⭕️
        - 명확 ↑
        - 가독성 ↑
    - 단점
        - 평생 그 포맷에 얽매이게 된다.
        - 유연성 ↓
            - 프로그래머들은 명시된 포맷에 맞춰 파싱하고, 객체를 만들고, 데이터로 저장하는 코드를 작성하게 된다.
            - 조금이라도 포맷이 변경되면 기존의 코드들은 엉망이 된다.
- 명시하지 않는 경우
    
    ```java
    /**
     * 이 약물에 대한 대략적인 설명을 반환한다.
     * 다음은 이 설명의 일반적인 형태이나,
     * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
     * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
     */
    @Override
    public String toString() { ... }
    ```
