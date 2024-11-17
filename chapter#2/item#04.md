# 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

이 장에서는 자바에서 인스턴스화를 막는 방법을 설명합니다.

### 인스턴스화를 막는 이유

- 정적 메서드와 정적 필드만을 담는 클래스를 만들고 싶은 경우가 있다.
- `java.lang.Math`, `java.util.Arrays`가 그런 성격을 지닌 대표적인 예시이다.
- 이런 유틸리티 클래스는 인스턴스로 만들어 쓰기 위한 목적이 아니다.
- 하지만, 생성자를 명시하지 않으면 컴파일러는 자동으로 기본 생성자를 만들기에, 인스턴스화가 가능하다.
- 사용자는 이 생성자가 자동으로 생성된 것인지 아닌지 구분할 수 없으며, 설계자의 의도를 오해할 수 있다.

### 인스턴스화를 막는 방법

- private 생성자를 추가해라.
    
    ```java
    public class UtilityClass {
    	// 기본 생성자가 만들어지는 것을 막는다.(인스턴스화 방지용)
    	private UtilityClass() { ... }
    }
    ```
    
    - 의도를 명확히 하기위해, 위와 같은 주석을 달아주도록 하자.