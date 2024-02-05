# 멤버 클래스는 되도록 static으로 만들라

## 중첩 클래스
중첩 클래스(nested class)란 클래스 내부에 클래스가 있는 형태의 클래스이다.
* 중첩 클래스는 자신을 감싼 바깥 클래스에서만 사용되어야 한다.
* 중첩 클래스가 바깥 클래스 이외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
* 중첩 클래스를 사용함으로써 불필요한 노출을 줄여 캡슐화를 할 수 있고 가독성 좋고 유지보수하기 좋은 코드를 작성할 수 있다.

## 중첩 클래스의  종류
1. 정적 멤버 클래스
2. (비정적) 멤버 클래스
3. 익명 클래스
4. 지역 클래스


## 정적 멤버 클래스
* 클래스 내부에 static으로 선언된 클래스다.
* 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근 가능. 그 외는 일반 클래스와 똑같다.
* private으로 선언시 바깥 클래스에서만 접근 가능하다.
* enum타입은 static을 명시하지 않더라도 static으로 취급된다.
```java
public class Animal {
  private String name = "cat";
  
  public enum Kinds {
    MAMMALS, BIRDS, FISH, REPTILES, INSECT
  }

  private static class PrivateSample {
    private int temp;

    public void method() {
      Animal outerClass = new Animal();
      System.out.println("private" + outerClass.name);
    }
  }

  public static class PublicSample {
    private int temp;

    public void method() {
      Animal outerClass = new Animal();
      System.out.println("public" + outerClass.name);
    }
  }
}
```
```java
public class Main {
    public static void main(String[] args) {
        Animal.PublicSample publicSample = new Animal.PublicSample();
        publicSample.method();
    }
}
```

## 비정적 멤버 클래스
* static이 붙지 않은 멤버 클래스
* 비정적 멤버 클래스의 인스턴스 메서드에서  클래스명.this 를 사용해 바깥 인스턴스의 메서드나 참조 가져올 수 있다
* 바깥 인스턴스 없이는 생성 불가능 하다.

```java
public class TestClass {
  private String name = "yeonlog";

  public class PublicSample {
    public void printName() {
      // 바깥 클래스의 private 멤버 가져오기
      System.out.println(name);
    }

    public void callTestClassMethod() {
      // 바깥 클래스의 메소드 호출하기
      TestClass.this.testMethod();
    }
  }

  public PublicSample createPublicSample() {
    return new PublicSample();
  }

  public void testMethod() {
    System.out.println("hello world");
  }
}
```

## 메모리 누수 가능성
비정적 멤버 클래스의 경우 바깥 클래스에 대한 참조를 가지고 있기 때문에 메모리 누수가 발생할 여지가 있다. 바깥 클래스는 더 이상 사용되지 않지만 내부 클래스의 참조로 인해 GC가 수거하지 못해서 바깥 클래스의 메모리 해제를 하지 못하는 경우가 발생할 수 있다.   
반면 정적 멤버 클래스의 경우는 바깥 클래스에 대한 참조 값을 가지고 있지 않기 때문에 메모리 누수가 발생하지 않는다.
## 익명 클래스
익명클래스란 이름이 없는 클래스로 바깥 클래스의 멤버가 아니며 사용시점에 선언 및 인스턴스가 생성된다.
```java
public class Calculator {
  private int x;
  private int y;

  public Calculator(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public int plus() {
    Operator operator = new Operator() {
      private static final String COMMENT = "더하기"; // 상수
      // private static int num = 10; // 상수 외의 정적 멤버는 불가능

      @Override
      public int plus() {
        // Calculator.plus()가 static이면 x, y 참조 불가
        return x + y;
      }
    };
    return operator.plus();
  }
}

interface Operator {
  int plus();
}
```
* instanceof 검사 및 클래스 이름이 필요한 작업 수행 불가
  * 선언과 동시에 인스턴스 생성하고 더이상 사용하지 않음.
  * 클래스 이름이 없어서 위 작업 불가.
* 내용이 길어지면 가독성이 떨어진다.
* 선언 지점에서만 인스턴스 생성 가능하다.
* 정적 문맥에서 static final 상수 외의 정적 멤버를 가질 수 없다.

## 지역 클래스
유효범위가 지역 변수와 같고, 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있고, 정적 멤버는 가질 수없고, 짧게 작성해야 한다.
```java
public class TestClass {
    private int number = 10;

    public TestClass() {
    }

    public void foo() {
        // 지역변수처럼 선언
        class LocalClass {
            private String name;
            // private static int staticNumber; // 정적 멤버 가질 수 없음

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                // 비정적 문맥에선 바깥 인스턴스를 참조 가능
                // foo()가 static이면 number에서 컴파일 에러
                System.out.println(number + name);
            }
        }
        LocalClass localClass1 = new LocalClass("local1"); // 이름이 있고
        LocalClass localClass2 = new LocalClass("local2"); // 반복해서 사용 가능
    }
}
```

## 결론
익명클래스를 사용해야 하는 경우나 바깥 인스턴스에 접근할 일이 없으면 가능한 정적 멤버 클래스를 사용하자

