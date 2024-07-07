# 객체 생성과 파괴

클래스의 인스턴스를 얻는 수단으로는 2가지가 있다.  
  1) public 생성자
  ```java
  public ClassName (boolean b)
  ```  
  3) static factory method  
  ```java
  public static Boolean valueOf(boolean b)
  ```

* `static factory method` 의 장점

#### 1. 객체의 특성을 쉽게 묘사할 수 있다.
 
	ex ) BigInteger vs BigInteger.probablePrime
  
#### 2. 인스턴스의 라이프 사이클을 통제할 수 있다.
   
   > 인스턴스 통제 클래스란?  
   > 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제

   싱글톤 패턴 가능, 인스턴스화 불가  
   동치인 인스턴스가 단 하나뿐 임을 보장할 수 있다.  
   <!-- 하나이면 뭐가 좋은데? -->

   `플라이웨이트 패턴` 참고
    
#### 3. 반환 객체의 클래스를 선택할 수 있다.

  구현 클래스를 공개하지 않고 객체를 반환할 수 있다.
  -> API를 작게 유지 가능  
  <!--  API 가 작으면 뭐가 좋지? -->

  JAVA 8 이전 : 인터페이스에 정적 메서드 선언 불가 -> 인스턴스화 불가 동반 클래스가 필요
  JAVA 8     : 인터페이스에 정적 메서드 선언 가능
                -> 동반 클래스에 public 멤버를 인터페이스에 선언
  JAVA 9     : 인터페이스에 private 정적 메서드 선언 가능 (정적 필드, 정적 멤버 클래스는 public)

#### 4. 다른 클래스의 객체를 반환할 수 있다.


  


#### 5. statc factory method를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.


* `static factory method` 의 단점

#### 1. 추가적인 public, protected 생성자가 필요하다.


#### 2. 프로그래머가 static factory method의 인스턴스화 방법을 알아내야한다.
  Java doc 에서 관리 안해줌.


* `static factory method` 의 명명 방식



<!--  정적 팩토리 메서드가 더 좋은 경우는 언제고 생성자가 더 좋은 경우는 언제인지 봐야겠다.   -->






