# 생성자에 매개변수가 많다면 빌더를 고려하라

**정적 팩터리와 생성자**
정적 패터리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점.

```java
public class Company extends BaseTimeEntity {  
    private Long id;  
    private String name;  
    private String homepageUrl; 
    private String industry;  
    private String logoUrl;  
    private Year foundedDate;  
    private String city;  
}
```

위 코드로 예시를 들면
이 클래스의 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다고 합니다.

이렇게 점층적 생성자 패턴도 쓸 수는 있지만, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기가 어렵다는 단점이 있습니다.

**setter 메서드를 활용한 자반빈즈 패턴**  
set을 활용하면 코드가 길어지긴 했지만 인스턴스를 만들기 쉽고 읽기 쉬운 코드가 됩니다.

하지만 자바빈즈의 심각한 단점은 객체 하나를 만들려면 메서드를 여러 개 호출해야하 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다는 점입니다. 

책에서 일관성이 무너진 상태라는게 이해가 잘 되지 않아서 인터넷에 찾아보니 객체의 모든 필드가 초기화 되어있지 않은 상태를 포함하는 것이라고 합니다.
https://okky.kr/questions/1259979
https://cat-alan3.tistory.com/15

**빌더 패턴 - 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비**  
>클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다.
>그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다**  
https://velog.io/@minwoorich/%EC%A0%9C%EB%84%A4%EB%A6%AD-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%9E%AC%EA%B7%80%EC%A0%81-%ED%83%80%EC%9E%85-%ED%95%9C%EC%A0%95
재귀적 타입 한정...?

공변 반환 타이핑을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.
공변 반환 타이핑에 대한 인터넷 글.
https://bperhaps.tistory.com/entry/%EA%B3%B5%EB%B3%80%EB%B0%98%ED%99%98-%ED%83%80%EC%9D%B4%ED%95%91
https://amenable.tistory.com/54
```java
Animal animal = new Cat() // 이런 것이 가능하다고 알고 있으면 될 듯 합니다
```

빌더 패턴에 장점만 있는게 아니라 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개이상은 되어야 값어치를 한다고 합니다.

저같은 경우엔 스프링에선 @Builder를 활용해서 많이 쓰고있습니다
![[Pasted image 20241117214405.png]]
