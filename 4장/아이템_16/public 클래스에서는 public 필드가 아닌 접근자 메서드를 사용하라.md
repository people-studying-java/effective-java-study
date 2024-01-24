> # public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
> ## public 필드를 왜 사용하면 안될까?
> 1. 데이터 필드에 직접 접근할 수 있으므로 캡슐화의 이점을 제공하지 못한다.
> 2. 불변식을 보장하지 않는다.
> 
> ```java
> public class Point {
>   public double x;
>   public double y;
> }
> ```
>
> ## 접근자와 변경자 메서드를 사용하여 캡슐화 하자
> 클래스 필드의 접근제한자를 private로 설정하고 public get/set 메소드를 생성하여 해당 메서드를 통해서만 값에 접근할 수 있게 수정하는게 올바른 방법이다. 
> ```java
> public class Point {
>   private double x;
>   private double y;
> 
>   public double getX(){
>       return x;
>   } 
> 
>   public double getY(){
>       return y; 
>   }
> 
>   public void setX(double x){
>       this.x=x;
>   }
>   public void setY(double y){
>       this.y=y;
>   }
> 
> }
> ```
> ## public 필드를 반드시 사용해야 한다면
> package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 문제가 없다.
> ```java
> public class ColorPoint {
> 
>   private static class Point{
>       public double x;
>       public double y;
>   }
> 
>   public Point getPoint(){ // 클라이언트 코드가 클래스 내부에 묶인다.
>         Point point = new Point(); //point에 접근하기 위해 생성자를 또 만들어야함..
>         point.x = 1.2; 
>         point.y = 5.3; 
> 
>         return point; 
>   }
> 
> }
> ```
>
> ## public 필드를 사용한 라이브러리
> public 필드를 사용한 대표적인 라이브러리 중 하나인 Dimension 클래스는 jdk1.0버전부터 사용되었지만 아직도 필드 공개로 인해 발생한 심각한 성능문제를 해결하지 못했다.
> ```java
> @Transient
> public Dimension getSize() {
>   return new Dimension(width, height); // 객체의 불변성을 위해 새로운 인스턴스를 다시 생성함
> }
>  
> ```
> 
> ## 결론
> public 필드를 사용하지 말자.