> # 자원 해제의 필요성
> > 자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야하는 자원이 많다 (InputStream,OutputStream,java.sql.Connection)   
> > 자원을 제때 close하지 않으면 예측할 수 없는 성능 문제를 야기하게된다.
> > > * 자원해제가 필요한 상당수의 라이브러리에서 최소한의 안전망으로 finalizer를 활용하고 있지만 안전하지 않다.(아이템 8 참고)
> # try-finally
> > 자원 해제를 위한 전통적인 방식으로 사용되었다.
> > ``` java 
> > public static String inputString() throws IOException {
> >   BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
> >    try {
> >        return br.readLine();
> >    } finally {
> >        br.close();
> >     }
> >   }
> > ```
> # try-finally의 문제점
> > ### 1. 코드가 복잡해 진다.
> > 해제해야할 자원이 많아질 경우 아래의 코드 처럼 복잡한 코드가 된다.
> > ``` java
> > public static void inputAndWriteString() throws IOException {
> >   BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
> >   try {
> >         BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
> >         try {
> >               String line = br.readLine();
> >               bw.write(line);
> >         } finally {
> >               bw.close();
> >         }
> >   } finally {
> >         br.close();
> >   }
> > }
> > ```
> > ### 2. 예외 발생에 대한 확인이 어렵다.
> > ``` java
> >     public static void main(String[] args) {
> >         try {
> >             check();
> >         } catch (Exception e) {
> >             e.printStackTrace();
> >         }
> >     }
> > 
> >     public static void check() {
> >         try {
> >             throw new IllegalArgumentException();
> >         } finally {
> >             throw new NullPointerException();
> >         }
> >     }
> > }
> > 
> > ```
> > 실행 결과 :   
> > java.lang.NullPointerException   
> > at Application.check(Application.java:14)  
> > at Application.main(Application.java:4)   
> > 위 코드를 실행하게 되면 try 안에서 발생한 예외는 출력되지 않는다.
> # try-with-resources
> try-finally 방식의 단점을 보완하기 위해 자바 7 버전부터는 try-with-resources가 도입되었다. try-with-resources를 사용하기 위해서는 사용하는 자원이 AutoCloseable 인터페이스를 구현해야 한다.
> ### try-with-resources 사용한 자원 해제
> 
> > ```
> > try (AutoCloseable 구현한 클래스){
> >     로직...
> > }
> > ```
> # try-with-resources의 장점
> > ## 1. try-finally를 코드와 비교하여 훨씬 간결하다
> > ``` java
> >   BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
> >   try {
> >         BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
> >         try {
> >               String line = br.readLine();
> >               bw.write(line);
> >         } finally {
> >               bw.close();
> >         }
> >   } finally {
> >         br.close();
> >   }
> > ```
> > ``` java
> > try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
> >      BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out))) {
> >     String line = br.readLine();
> >     bw.write(line);
> > }
> > ```
> > ## 2. 예외 발생해 대한 확인이 수월하다.
> > ```java
> > public class Application {
> > 
> >     public static void main(String[] args) {
> >         try {
> >             check();
> >         } catch (Exception e) {
> >             e.printStackTrace();
> >         }
> >     }
> > 
> >     public static void check() throws Exception {
> >         try (IllegalArgumentExceptionThrower thrower = new IllegalArgumentExceptionThrower()) {
> >             throw new NullPointerException();
> >         }
> >     }
> > 
> >     static class IllegalArgumentExceptionThrower implements AutoCloseable {
> >         @Override
> >         public void close() throws Exception {
> >             throw new IllegalArgumentException();
> >         }
> >     }
> > }
> > ```
> > 실행 결과
> > 
> > java.lang.NullPointerException  
> >     at Application.check(Application.java:17)   
> >     at Application.main(Application.java:9)   
> >     Suppressed: java.lang.IllegalArgumentException   
> >         at Application$IllegalArgumentExceptionThrower.close(Application.java:24)   
> >         at Application.check(Application.java:16)   
> >         ... 1 more   
> # 결론
> 자원해제를 할떄는 try-finally를 사용하지 말고 반드시 try-with-resources 를 사용하자