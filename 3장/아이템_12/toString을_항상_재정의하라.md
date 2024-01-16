# toString을 항상 재정의하라

모든 클래스의 최상위 클래스인 'Object' 클래스가 가진 'toString' 메소드를 확인할 수 있다.
```java
public String toString() {
    return getClass().getName() + "@" +Integer.toHexString(hashCode());
} 
```
즉, 모든 클래스는 'toString' 메소드를 이용할 수 있으며, 일반적인 규약으론 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야한다.  
또한 '모든 하위 클래스에서 이 메서드를 제정의하라'는 규약도 포함되어있다.  
> toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.
> 가령 map 객체를 출력했을 경우 {Jenny=PhoneNumber@adbbd} 보단 {Jenny=707-867-5309}이 더 유용함을 알 수 있따.

***
## 포맷 문서화 여부를 결정하자
선택이지만, 표준적이고,명확하고,사람이 읽기 편안한 문서화를 지향하자  
포맷을 명시하리고 했다면, 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 '정적 팩터리' or '생성자'를 함께 제공해주자
ex
```java
public class toStringExp {
    @Builder
    class Address {
        private final String detail;
        private final String street;
        private final int zipCode;
        
        public Address valueOf(String address) {
            String[] split = address.split(", ");
            return Address.builder()
                    .detail(split[0])
                    .street(split[1])
                    .zipCode(Integer.parseInt(split[2]))
                    .build();
        }

        @Override
        public String toString() {
            return String.format("%s, %s, %d",detail,street,zipCode);
        }
    }
}
```
문서화 장점
- 값을 입출력에 그대로 사용 가능
- CSV 같이 데이터 객체로 저장이 가능하다

문서화 단점
- 해당 포맷에 영구적으로 얽매이게됨

> 포맷을 문서화 하지 않은 경우, 추가,수정에 대한 유연성을 확보할 수 있다.  
***
포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된  
정보를 얻을 수 있는 API를 제공하자(Getter)

***
### toString을 재정의하지 않아도 되는 경우
- 정적 유틸리티 클래스
  - 객체를 만들어서 사용하지 않음(표현할 상태가 없다)
- enum
  - 이미 알기 쉬운 정보로 잘 처리되어있음
***
## 참고
JPA에  순환참조를 조심해야한다.
```java
@ToString
public class Team {
    @Id
    @GeneratedValue
    private long id;
    private String teamName;
    @OneToMany(mappedBy = "user")
    private Set<Member> users;
}
```
```java
@ToString
public class Member {
    @Id
    @GeneratedValue
    private long id;
    private String userName;
    @ManyToOne
    private Team team;
}
```
위 경우 Member <-> Team 간의 toString() 호출되어 스택오버플로우 발생
- @ToString(exclude = "team")
- toString() 직접 구현