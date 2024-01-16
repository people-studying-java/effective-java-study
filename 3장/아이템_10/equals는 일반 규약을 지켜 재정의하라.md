> # equals는 일반 규약을 지켜 재정의하라
> equals 메소드는 자주 사용된다. 경우에 따라 equals 메소드를 재정의 하여 사용하는데 잘못된 방식으로 재정의 할 경우 큰 피해가 생긴다.   
> 재정의가 필요한 상황과 기본 equlas 메소드를 사용해야 하는 상황을 알아보자
> ## equals 메소드
> equals 메소드는 Object클래스에 정의되어 있는 메소드로 두 객체의 주소값을 비교하여 동등한지 판단한다.
> ```java
> public boolean equals(Object obj) {
> return (this == obj);
> }
> ```
> ## 재정의 없이 eqauls 메소드를 사용하는 경우
> 1. 값을 표현하는게 아닌 동작하는 개체를 표현하는 클래스일 경우 (Thread 클래스)
> 2. 논리적 동치성(주소값이 아닌 값 자체, 필드 값 자체가 같은것)을 검사할 일이 없는 경우
> 3. 상위 클래스에서 재정의한 equals를 수정없이 계속 사용해도 되는 경우
>    * Set 구현체
>    * List 구현체
>    * Map 구현체
> 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 경우
> 5. 인스턴스가 둘 이상 만들어지지 않는 경우 (Enum이나 싱글턴 등)
>
> ## equals 메소드를 재정의 하는 경우
> 객체의 논리적 동치성을 비교해야하는데 상위 클래스에서 정의된 equals메소드가 논리적 동치성을 비교하지 않도록 정의되어 있는 경우이다.
> ## equals 메소드 재정의 규약
> equals 메소드를 재정의 할떄는 아래의 4가지 규칙을 반드시 지켜야 한다.   
> 클래스들은 자신에게 전달된 객체가 equals 규약을 지킨다고 가정하고 동작한다.
> ### 1. 반사성
>   * null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야 한다.
>   * 자기 자신과 같아야 한다.
> ### 2. 대칭성
>   * null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.
>   * 서로에 대한 동치 여부가 같아야 한다.
> ```java
> public final class CaseInsensitiveString {
> 
>     private final String str;
>     
>     public CaseInsensitiveString(String str) {
>         this.str = Objects.requireNonNull(str);
>     }
>     
>     @Override
>     public boolean equals(Object o) {
>         return str.equalsIgnoreCase((String) o);
>     }
> }
> ```
> ```java
>   void 동치성_테스트() {
>   CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
>   String test = "test";
>   System.out.println(caseInsensitiveString.equals(test)); // 재정의한 equals를 사용하여 대소문자 구분 없이 값 비교 당연히 true
>   System.out.println(test.equals(caseInsensitiveString)); // 문자열의 equals를 사용 당연히 false
>   }
> ```
> ### 3. 추이성
>   * null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true이다.
>   * a==b , b==c , a==c
> ```java
> public class Point {
> 
>     private final int x;
>     private final int y;
>     
>     public Point(int x, int y) {
>         this.x = x;
>         this.y = y;
>     }
>     
>     @Override
>     public boolean equals(Object o) {
>         if (!(o instanceof Point)) {
>             return false;
>         }
>         Point p = (Point) o;
>         return this.x == p.x && this.y == p.y;
>     }
> }
> ```
> ```java
> public class ColorPoint extends Point {
> 
>     private final Color color;
> 
>     @Override
>     public boolean equals(Object o) {
>         if (!(o instanceof Point)) {
>             return false;
>         } 
> 
>         // o가 일반 Point이면 색상을 무시햐고 x,y 정보만 비교한다.
>         if (!(o instanceof ColorPoint)) {
>             return o.equals(this);
>         }
> 
>         // o가 ColorPoint이면 색상까지 비교한다.
>         return super.equals(o) && this.color == ((ColorPoint) o).color;
>     }
> }
> ```
> ```java
> void 추이성_테스트() {
> ColorPoint a = new ColorPoint(2, 3, Color.RED);
> Point b = new Point(2, 3);
> ColorPoint c = new ColorPoint(2, 3, Color.BLUE);
> 
>     System.out.println(a.equals(b)); // true
>     System.out.println(b.equals(c)); // true
>     System.out.println(a.equals(c)); // false
> }
> ```
> ### 4. 일관성
> * null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
> ### 5. null-아님
> * null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false
> ```java
> @Override
> public boolean equals(Object o) {
>   // 우리가 흔하게 인텔리제이를 통해서 생성하는 equals는 다음과 같다.
>   if (o == null || getClass() != o.getClass()) {
>   return false;
>   }
> 
>   // 책에서 추천하는 내용은 null 검사를 할 필요 없이 instanceof를 이용하라는 것이다.
>   // instanceof는 두번째 피연산자(Point)와 무관하게 첫번째 피연산자(o)거 null이면 false를 반환하기 때문이다. 
>   if (!(o instanceof Point)) {
>     return false;
>   }
> }
> ```
>
>
> # 결론
> 논리적 동치성을 확인하는 경우가 아닐떄는 eqauls 메소드를 재정의 하지 말자.   
> 만약 재정의 할 경우에는 규칙에 위배되지 않게 작성하자.