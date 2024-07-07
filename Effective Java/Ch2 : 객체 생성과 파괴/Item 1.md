# Item 1 - 생성자 대신 정적 팩터리 메서드를 고려하기


클래스의 인스턴스를 얻는 수단으로는 2가지가 있다.  
  1) public 생성자
  ```java
  public ClassName (boolean b)
  ```  
  2) static factory method  
  ```java
  public static Boolean valueOf(boolean b)
  ```

  > **factory** 란? 
  >    클래스의 객체를 생성하는 역할을 서브 클래스에 위임하는 것

### static factory method 의 장점
---

#### 1. 이름으로 객체의 특성을 쉽게 묘사할 수 있다.
 
	ex) BigInteger vs BigInteger.probablePrime  

  
#### 2. 인스턴스의 라이프 사이클을 통제할 수 있다.
   
   > **인스턴스 통제 클래스**란?  
   >  언제 어느 인스턴스를 살아 있게 할지를 철저히 통제  

   싱글톤 패턴 가능, 인스턴스화 불가  
   동치인 인스턴스가 단 하나뿐 임을 보장할 수 있다.  
   <!-- 하나이면 뭐가 좋은데? -->

   참고. `플라이웨이트 패턴` 
    
#### 3. 반환 객체의 클래스를 선택할 수 있다.

    구현 클래스를 공개하지 않고 객체를 반환할 수 있다.
    -> API를 작게 유지 가능  
  <!--  API 가 작으면 뭐가 좋지? -->

    JAVA 8 이전 : 인터페이스에 정적 메서드 선언 불가 -> 인스턴스화 불가 동반 클래스가 필요
    JAVA 8     : 인터페이스에 정적 메서드 선언 가능
                  -> 동반 클래스에 public 멤버를 인터페이스에 선언
    JAVA 9     : 인터페이스에 private 정적 메서드 선언 가능 (정적 필드, 정적 멤버 클래스는 public)

#### 4. 다른 클래스의 객체를 반환할 수 있다.

    EX ) EnumSet Class : static factory만 제공함.
          원소의 수에 따라 RegularEnumSet / JumboEnumSet 인스턴스를 반환한다.
          => 하지만, 클라이언트는 두 클래스의 존재를 모름

          다음 릴리스에 해당 객체를 수정 / 삭제해도 문제가 발생하지 않는다. => 유지보수 용이

#### 5. static factory method를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.  

    서비스 제공자 프레임워크(ex. JDBC)를 만들 때 주로 사용된다.  
    
    구성 : 서비스 인터페이스(Connection)
          제공자 등록 API(DriverManager.registerDriver)
          서비스 접근 API(DriverManager.getConnection)
    
    이때, 서비스 접근 API를 사용할 때 클라이언트가 구현 조건을 명시하지 않은 건에 대해 
    default값을 반환함.
    
  참고. `Bridge 패턴` `의존 객체 주입`








### `static factory method` 의 단점
---

#### 1. 추가적인 public, protected 생성자가 필요하다.

    컬랙션 프레임워크의 유틸리티 구현 클래스는 상속할 수 없다.
    (ex. Collections.sort() ..)
    


#### 2. 프로그래머가 static factory method의 인스턴스화 방법을 알아내야한다.
  Java doc 에서 관리를 하지 않기 때문에 **개발자가 따로 인스턴스화 방법을 알아내야 한다.**



###  `static factory method` 의 명명 방식
---

* from : 해당 타입으로의 형변환 메서드
  `ex) Date d = Date.from(instance);`

* of : 적합한 타입의 인스턴스를 반환하는 집계 메서드
  `ex) Set<Rank> cards = EnumSet.of(J, Q, K);`

* valueOf : from과 of의 더 자세한 버전
 <!-- 어떻게 더 자세한지...? -->
  ex) BigInteger p = BigInteger.valueOf(Integer.MAX_VALUE);

* instance || getInstance : 매개변수로 명시한 인스턴스 반환 (같은 인스턴스 보장 X)
  ex) Pizza pisa = Pizza.getInstance(options);

* create || newInstance : 새로운 인스턴스를 생성해 반환함을 보장
  ex) Object newArray = Array.newInstance(classObject, arraylen);

* getType : getInstance와 같지만, 다른 클래스에서 팩터리 메서드를 정의할 때 사용
  ex) FileStore fs = Files.getFileStore(path);

* newType : newInstance와 같지만, 다른 클래스에서 팩터리 메서드를 정의할 때 사용
  ex) BufferedReader br = Files.newBufferedReader(path);
  <!-- 구체적인 상황은 ? -->

* type : getType 과 newType 의 축약형
  ex) List<Complaint> litany = Collections.list(legacyLitany);




<!--  정적 팩토리 메서드가 더 좋은 경우는 언제고 생성자가 더 좋은 경우에 대해 자세히.   -->






