## 싱글톤(Singleton)

- 인스턴스가 하나임을 보장하는 디자인 패턴
- 처음 1개의 인스턴스만을 생성한 후 같은 인스턴스사용  
ex) 함수같은 무상태(stateless)객체 or 설계상 유일한 시스템 컴포넌트
 
> statelss ? 공유하는 필드 변수가 없는 것
> 값의 변경X, 가능한 메서드를 이용한 read만 가능하도록!

상태있는(stateful) 객체를 여러클라이언트가 조회할때 발생하는 문제
```java
public class StatefulService {
	private int number; //상태 유지 필드

	public void order(int number);
		System.out.println("number= "+number);
		this.number = number;
	}
	public int getNumber() {
		return number;
	}
}
```
위 경우 여러 클라이언트에서 StatefulService를 호출할 경우 각각 number가 상이하다면 마지막에 호출된 number로 모든 StatefulService의 number가 업데이트

***
### 장점
- 클래스에 인스턴스가 하나만 있음을 확인,보증
- 메모리 영억에 할당되는 인스턴스가 하나 = 메모리 낭비하지 않을 수 있다
### 단점
- 전역적인 접근 ⇒ 다른 클래스 인스턴스와 결합도가 높아짐 ⇒ 단일 책임 원칙을 위배할 수도 있다(SRP)
- 멀티 쓰레드 환경에서 동기화 처리 X 경우 → 다중 인스턴스 생성 문제 가능성
- 싱클톤 인스턴스의 단위 테스트 어려움…?(무슨뜻인지 하하)

***

## 생성 방식(보통 2가지)
공통 : 생성자는 private / 유일한 인스턴스 접근 수단 public static 맴버  

1. 단순한 구현(Eager Initialization Singleton)
```java
    public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    public vod leaveTheBuilding() { ... }
```
private 생성자 = Elvis.INSTANCE를 초기화 할때 딱 한번 호출  
or ClassLoading 시점에 싱글톤 객체는 생성된다.
- new를 통한 초기화
- class.forName()과 함께 Reflection을 사용하는 경우
- 해당 클래스의 static메소드 실행
- 해당 클래스의 static 필드에 값 할당 혹 사용(단,static final 제외)
- Top-Level class인데, assert문 사용하는 경우

2. 정적 팩터리 메서드 사용(Lazy Initializtion)  
   객체가 필요한 경우에만 초기화하고, Singleton 객체를 반환받을 수 있는 진입점(static 메소드) 제공하는 방식
```java
public class Singleton {
  public static Singleton INSTANCE;
  
  private Singleton() {
    System.out.println("Simple singleton instance initialized.");
  }
  
  public static Singleton getInstance(){
    if(INSTANCE == null)
      INSTANCE = new Singleton();
    return INSTANCE;
  }
}
```
명시적 getInstance 메소드 호출 X ⇒ 객체 초기화X = Eager Initialzation 단점 보완  
But! Multi Thread 환경 문제점 존재 => 동기화 키워드 사용(synchronized)

### 두 방식의 차이점
#### Eager
- 싱글톤임을 API에 명확히 확인 가능 ⇒ public static 필드가 final !!
- 간결함

#### Lazy
- API 변경 없이 싱글톤 형태 변경 가능(return 부분 new INSTANCE;)
- 원한다면 정적 팩터리 제네릭 싱글턴 팩터리로 만들 수 있다..?
- 정적 팩토리의 메소드 참조를 공급자로 사용할 수있다

***
### 역직렬화로 깨지는 싱글톤?
> JAVA-Serialize : JVM의 힙 메모리에 있는 객체 데이터를 바이트 스트림 형태로 바꿔 외부 파일로 보낼 수 있는 기술  
> JAVA-Deserialize : 외부로 내보낸 직렬화 데이터를 다시 읽어들여 자바 객체로 변환하는 기술
 ```java
public class test {
    public static void main(String[] args) throws
            IOException,
            ClassNotFoundException {
        Singleton singleton1 = Singleton.getInstance();

        String fileName = "singleton.obj";

        // 직렬화
        ObjectOutputStream out = new ObjectOutputStream(new BufferedOutputStream(new FileOutputStream(fileName)));
        out.writeObject(singleton1);
        out.close();

        // 역직렬화
        ObjectInputStream in = new ObjectInputStream(new BufferedInputStream(new FileInputStream(fileName)));
        Singleton singleton2 = (Singleton) in.readObject();
        in.close();

        System.out.println("singleton1 == singleton2 : " + (singleton1 == singleton2));
        System.out.println(singleton1);
        System.out.println(singleton2);
    }
}
```
결과  
```
singleton1 == singleton2 : false
Singleton@6d6fe28
Singleton@378bf509
```

이유  
역직렬화 자체가 보이지 않는 생성자로서 역할을 수행하기 때문!  
= 인스턴스를 또 만들어서 진행됨..!  
= 메모리 이점이 없어진다.
#### 해결방안
```
private Object readResolve() {
	return INSTANCE;
}
```

### 리플렉션으로 깨지는 싱글톤?
런타임에 클래스의 동작을 조작하는 기법
```java
public class Test {
   public static void main(String[] args) throws NoSuchMethodException, InstantiationException, IllegalAccessException, InvocationTargetException {

      Singleton singleton = Singleton.INSTACE;

      Constructor<Singleton> consructor = Singleton.getClass().getDeclaredConstructor(new Class[0]);

      // 2. 생성자가 private 이기 때문에 외부에서 access 할 수 있도록 true 설정
      consructor.setAccessible(true);

      // 3. 가져온 생성자를 이용해 인스턴스화 한다
      Singleton singleton1 = consructor.newInstance();
      Singleton singleton2 = consructor.newInstance();

      System.out.println("singleton1 == singleton2 : " + (singleton1 == singleton2));
      System.out.println(singleton1);
      System.out.println(singleton2);
   }
}
```
결과  
```
singleton1 == singleton2 : false
Singleton@6d6fe28
Singleton@378bf509
```
해결 방안  
```
public enum Singleton {
	INSTANCE;
	int value;
	public int getValue() {
		return value;
	}
}
```