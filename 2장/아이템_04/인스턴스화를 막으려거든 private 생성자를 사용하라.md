# 인스턴스화를 막으려거든 private 생성자를 사용하라

정적(static) 메서드 or 필드만 담은 클래스 = 객체지향과 거리가 멀지만 쓰임새가 있다  

### 유틸리티 클래스
인스턴스 메서드와 변수를 일정 제공하지 않는 클래스  
'비슷한 기능의 메소드와 상수를 모아서 캡슐화한것'
> 객체지향과 거리가 먼 이유?
> 객체지향 프로그래밍 원칙 '한 객체가 지니고 있는 데이터들은 외부에서 함부로 접근하여 수정 할 수 없도록'  
> 각 객체의 데이터들이 캡슐화되어야한다는 객체지향 프로그램 원칙에 위배됨

예시
> java.lang.Math와 java.util.Arrays 처럼 기본 타입 값이나 배열 관련 메서드 모아놓음  
> java.util.Collections 처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드 모아놓음  
> final 클래스와 관련한 메서들 모아놓음

```java 
public class Decision {
    private static String answer;
    private static String guessAnswer;
    
    public static boolean isCorrect() {
        return answer.equls(guessAnswer);
    }
}
```
위 경우 인스턴스를 만들어 쓰려고 설계한것이 아님에도 불구하고, 생성자 명시가 없다면  
컴파일러가 자동으로 기본생성자(public 생성자)를 만들어버린다.
```java 
Decision decision = new Decision();
boolean isCorrect1 = decision.isCorrect();  // 잘못 사용
boolean isCorrect2 = Decision.isCorrect();  // 바른 사용

```

***
### 인스턴스화 막는방법
> 추상 클래스로 정의한다?  
> 하위 클래스(상속 받는)에서 인스턴스화가 일어날 수 있다.
 ```aidl 
 abstract class Decision {
   ...
 }
 class SubDecision extends Decision {
   ...
 }
 public static void main(String[] args) {
   // Decision decision = new Dcision();
   // 불가능 - java.lang.Error: Unresolved compilation problem
   SubDecision subDecision = new SubDecision();
 ```
***
## private 생성자를 추가하자
```java
public class Decison {
    private Decison() {
        throw new AssertionError();
        // 클래스 안에서 실수라도 생성자를 호출하지 않도록 위함
    }
    public static void main(String[] args) {
        Decison decison = new Decison();
        // 예외 발생
    }
}
```

- 생성자가 명시적으로 존재하기때문에 컴파일러가 기본생성자 만듬X, private 생성자라서 접근 불가
- 상속이 불가능 = 모든 생성자는 상위 클래스의 생성자를 호출해야하는데 private임
- 생성자가 존재하는데 호출안되기때문에 주석을 달아주자!
***
#### 참고사이트
[1] https://velog.io/@alkwen0996/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-%EC%95%84%EC%9D%B4%ED%85%9C4-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%ED%99%94%EB%A5%BC-%EB%A7%89%EC%9C%BC%EB%A0%A4%EA%B1%B0%EB%93%A0-private-%EC%83%9D%EC%84%B1%EC%9E%90%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC  
[2] https://lovethefeel.tistory.com/133?category=957374  
[3] https://devyihyun.tistory.com/111
