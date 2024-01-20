# Comparable을 구현할지 고려하라

- Comaprable 인터페이스의 유일한 메소드 compareTo를 확인할 수 있다.  
> Object와 2가지를 제외하면 성격이 동일하다
> 1. 단순 동치성비교에서 확장하여 순서까지 비교 가능
> 2. 제네릭함
>    * 동치성비교 : 두 객체가 같은지
>    * 제네릭하다 : 타입이 일반화 되어있다?

- Comparable을 구현 했다 = 해당 클래스의 인스턴스들에겐 자연적인 순서(natural order)가 존재함
```java
    Arrays.sort(a);
```
- 예) 비교를 활용하는 정렬된 컬렉션 TreeSet,TreeMap / 정렬 알고리즘 활용하는 유틸리티 클래스 Collection,Arrays
  - TreeSet(add시 내부적 compare()활용)
***
### 규약
순서 비교시 해당 객체가 주어진 객체보다 작으면 음의 정수(-1) / 같으면 (0) / 크면 (+1) 반환  
비교할 수 없는 타입의 경우 ClassCastException
> 참고
> - x.compareTo(y) == - y.compareTo(x)
>   - 두 객체 참조 순서 바꿔도 동일한 결과 반환
> - 추이성 
> (x.compareTo(y) > 0 && y.compareTo(z) >0) == x.compareTo(z) > 0
>   - x가 y보다 크며 y가 z보다 크면 x 는 z보다 크다
> - x.compareTo(y) == 0 (x.compareTo(z) == z.compareTo(z))
>   크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같음 
> - (x.compareTo(y) == 0) == x.equals(y)
>   - 선택사항(추천) 동치성 비교시 문제발생
```java
public class Pro {
    void example() {    //BigDecimal 정밀한 값
        BigDecimal number1 = new BigDecimal("1.0");
        BigDecimal number1 = new BigDecimal("1.00");
        
        Set<BigDecimal> set1 = new HashSet<>();
        set1.add(number1);
        set1.add(number2); // equals()로 원소 비교 => 2개
        
        Set<BigDecimal> set2 = new TreeSet<>();
        set2.add(number1);
        set2.add(number2); // compareTo()로 원소 비교 => 1개
    }
}
```
***
## compareTo() 작성 요령
- Comparable 을 implements시 타입을 받는 제네릭 인터페이스 == compareTo() 인수 타입은 컴파일 단계에서 정해짐
- 각 필드의 동치확인이 아닌 [순서] 비교
- Compareble을 구현하지 않은 필드 or 표준이 아닌 순서 비교시 Comparator(interface) 사용해서 직접 구현
- 관계연산자(<,>) 보단 박싱 기본 타입클래스의 정적 메서드 compare()을 이용
- 핵심 필드부터 비교해나가서 순서 결정시 곧장 반환
```java
public class Test {
    public int compareTo(PhoneNumber pn) {
        int result = Short.compare(areaCode, pn.areaCode);  //1st
        if(result == 0) {
            result = Short.compare(prefix, pn.prefix);      //2nd
            if(result == 0) {
                result = Short.compare(lineNum,pn.lineNum); //3rd
            }
        }
        return result;
    }
    // 동일한 Test 클래스에 comparable 을 implement 했을 경우
    private static final Cmparator<PhoneNumber> COMPARATOR = 
            comparingInt((PhoneNumber pn) -> pn.areaCode)
                    .thenComparingInt(pn -> pn.prefix)
                    .thenComparingInt(pn -> pn.lineNum);
    
    @Ovverride
    public int compareTo(PhoneNumber pn) {
        return COMPARATOR.compare(this,pn);
    }
    // 약간의 성능저하 발생(교제기준 기준 10%)
    // long,double,short은 Int용 , float은 double용
}
```

값의 차이를 기준으로 비교하는 코드시 단순비교시 정수 오버플로우나, 소수점 계산방식에 따른 오류 발생할수 있다.
```java
public class Exam {
    static Comparator<Object> hashCodeOrder = new Comparator<>() {
        public int compare(Object o1, Obejct o2) {
            return o1.hashCode() - o2.hascode();
        }
    };
    
    //
    return Integer.compare(o1.hashCode(),o2.hashCode());
    static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
}
```