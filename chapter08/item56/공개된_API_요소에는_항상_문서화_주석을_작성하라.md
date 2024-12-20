API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다.

## Javadoc
[How to Write Doc Comments](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)

자바 4 이후로는 갱신되지 않은 페이지지만, 그 가치는 여전하다.

자바 버전이 올라가며 추가된 중요한 자바독 태그로는 자바 5의 `@literal`, `@code`, 자바 8의 `@implSpec`, 자바9의 `@index`를 꼽을 수 있다.

API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.  
상속용으로 설계된 클래스가 아니라면  
- how가 아닌 what을 기술.
- 해당 메서드를 호출하기 위한 전제조건을 모두 나열.
- 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건 모두 나열.
- 전제조건은 `@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술.
- `@param`태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다.
- 부작용도 문서화. (시스템 상태에 어떠한 변화를 가져오는 것을 뜻함)
- 반환 타입이 `void`가 아니라면 `@return`태그를 달아야 함. (태그의 설명이 메서드 설명과 같을 때 생략 가능)

`@throws`태그의 설명은 `if`로 시작해 해당 예외를 던지는 조건을 설명하는 절이 뒤따른다.  
관례상 `@param`, `@return`, `@throws` 태그의 설명에는 마침표를 붙이지 않는다.
```java
/**  
 * Returns the element at the specified position in this list. * * <p>This method is <i>not</i> guaranteed to run in constant * time. In some implementations it may run in time proportional * to the element position. * * @param  index index of element to return; must be  
 *         non-negative and less than the size of this list * @return the element at the specified position in this list  
 * @throws IndexOutOfBoundsException if the index is out of range  
 *         ({@code index < 0 || index >= this.size()})  
 */E get(int index) {  
    return null;  
}
```
설명이 한글이라면 온전한 종결어미로 끝날 때 마침표를 써주는게 일관돼 보인다고 함.

자바독 유틸리티는 문서화 주석을 HTML로 변환하므로 문서화 주석안의 HTML 요소들이 최종 HTML 문서에 반영된다.

## {@code}
태그의 효과
1. 태그로 감싼 내용을 코드용 폰트로 렌더링
2. 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시
여러 줄로 된 코드 예시를 넣으려면 `{@code}`태그를 다시 `<pre>`태그로 감싸면 된다.  
`<pre>{@code ... 코드 ... }</pre>`형태로 쓰면 된다.

단, @ 기호에는 무조건 탈출문자를 붙여야 하니 문서화 주석 안의 코드에서 애너테이션을 사용한다면 주의!

영문화 주석에서 `this`는 호출된 메서드가 자리하는 객체를 가리킨다.

## {@implSpec}
클래스를 상속용으로 설계할 때는 자기사용 패턴에 대해서도 문서에 남겨 다른 프로그래머에게 그 메서드를 올바로 재정의하는 방법을 알려줘야 한다.

`@implSpec`은 해당 메서드와 하위 클래스 사이의 계약을 설명하여, 하위 클래스들이 그 메서드를 상속하거나 `super`키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야 한다.

```java
/**  
 * Returns true if this collection is empty. * * @implSpec This implementation returns {@code this.size() == 0}.  
 * * @return true if this collection is empty  
 */public boolean isEmpty() {  
    return false;  
}
```

## {@literal}
API 설명에 `<`, `>`, `&`등의 HTML 메타문자를 포함시키려면 특별한 처리를 해줘야 한다.

`@literal`태그는 HTML 마크업이나 자바독 태그를 무시하게 해준다.  
`{@code}`와 비슷하지만 코드 폰트로 렌더링하지는 않는다.
```java
/**  
 * A geometric series converges if {@literal |r| < 1}.  
 */
public void fragment() {  
}
```
여기서 `<`기호만 태그로 감싸줘도 되지만 그렇게 하면 코드에서의 문서화 주석을 읽기 어려워진다.

**문서화 주석은 코드에서건 변환된 API 문서에서건 읽기 쉬워야 한다는 게 일반 원칙!**

양쪽을 만족하지 못하겠다면 API 문서에서의 가독성을 우선.

---
각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주.

요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.

헷갈리지 않으려면 한 클래스 안에서 요약 설명이 똑같은 멤버가 둘 이상이면 안 된다. (다중 정의된 메서드가 있다면 특히 더 조심)

요약 설명에는 `.`마침표에 주의해야 한다.  
`A suspect, such as Colonel Mustard or Mrs. Peacock.` 으로 첫 문장이 주어지면
`A suspect, such as Colonel Mustard or Mrs.` 까지만 요약 설명이 된다.

요약 설명이 끝나는 판단 기준은 처음 발견되는 `{<마침표> <공백> <다음 문장 시작>}` 패턴의 `<마침표>`까지.  
`<다음 문장 시작>`은 `소문자가 아닌 문자`다.

좋은 해결책은 의도치 않은 마침표를 `{@literal}`로 감싸주는 것이다.
```java
/**  
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.  
 */
public enum Suspect {  
    MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE  
}
```

---
> 요약 설명이란 문서화 주석의 첫 문장이다.

⬆️오해의 소지가 있음.

주석 작성 규약에 따르면 요약 설명은 완전한 문장이 되는 경우가 드물기 때문이다.

- 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는(주어가 없는) 동사구여야 한다.
- 2인칭 문장이 아닌 3인칭 문장으로 써야 한다.
- 한편 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다.

## {@index}
자바 9부터는 자바독이 생성한 HTML 문서에 검색(색인) 기능이 추가되어 넓은 API 문서에서 찾아가는 것이 수월해졌다.

```java
/**  
 * This method complies with the {@index IEEE 754} standard.  
 */
public void fragment2() {  
}
```

![image](https://github.com/user-attachments/assets/14b37145-e29b-4c24-807e-1555df499696)


## 제네릭타입 문서화
문서화 주석에서 제네릭, 열거 타입, 애너테이션은 특별히 주의해야 한다.

**제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.**
```java
/**
 * A utility class for transforming collections.
 *
 * @param <T> the type of the elements in the input collection
 * @param <R> the type of the elements in the result collection
 */
public class Transformer<T, R> {
    private final Function<T, R> transformer;

    public Transformer(Function<T, R> transformer) {
        this.transformer = transformer;
    }

    /**
     * Transforms a collection of elements.
     *
     * @param input the input collection
     * @return a new collection with transformed elements
     */
    public Collection<R> transform(Collection<T> input) {
        return input.stream().map(transformer).toList();
    }
}
```

## 열거 타입 문서화
열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
```java
/**  
 * An instrument section of a symphony orchestra. 
 */
public enum OrchestraSection {  
    /** Woodwinds, such as flute, clarinet, and oboe. */  
    WOODWIND,  
  
    /** Brass instruments, such as french horn and trumpet. */  
    BRASS,  
  
    /** Percussion instruments, such as timpani and cymbals. */  
    PERCUSSION,  
  
    /** Stringed instruments, such as violin and cello. */  
    STRING;  
}
```
열거 타입 자체와 그 열거 타입의 `public`메서드도 물론이다.

설명이 짧다면 주석 전체를 한 문장으로 써도 된다.

## 애너테이션 문서화
**애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.**

필드 설명은 명사구로 한다.
```java
/**  
 * Indicates that the annotated method is a test method that * must throw the designated exception to pass. 
 */
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface ExceptionTest {  
    /**  
     * The exception that the annotated test method must throw     * in order to pass. (The test is permitted to throw any     * subtype of the type described by this class object.)     
     */    
	Class<? extends Throwable> value();  
}
```
애너테이션 타입의 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

---
패키지를 설명하는 문서화 주석은 `package-info.java`파일에 작성한다.

모듈 시스템을 사용한다면 모듈 관련 설명은 `module-info.java`파일에 작성하면 된다. (자바 9부터 지원)

**API 문서화에서 자주 누락되는 설명이** 두 가지가 있다.
스레드 안정성과 직렬화 가능성이다.
- 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
- 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.
- 자바독은 메서드 주석을 상속시킬 수 있다.
- `{@inheritDoc}`태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.

자바독은 프로그래머가 자바독 문서를 올바르게 작성했는지 확인하는 기능을 제공하며, 이번 아이템에서 소개한 권장사항 중 상당수를 검사해준다. (명령어가 있지만 자바 8부터는 기본으로 작동하므로 제외했습니다.)

[인텔리제이에서 checkstyle 적용](https://hbase.tistory.com/443)
