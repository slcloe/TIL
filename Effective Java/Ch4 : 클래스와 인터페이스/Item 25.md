# Item 25 - 톱레벨 클래스는 한 파일에 하나만 담기

#### 두 클래스가 한 파일에 정의되었을 때

* Utensil.java
```java
class Utensil {
  static final String NAME = "pan";
}

class Dessert {
  static final String NAME = "cake";
}
```

* Dessert.java
```java
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

* Main.java
```java
public class Main {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Desert.NAME);
  }
}
```

##### 결과
* `javac Main.java Dessert.java` : 컴파일 오류, Utensil, Dessert클래스를 중복 정의 했습니다.
	* Main.java를 먼저 컴파일하고, 그 안에서 Utensil 참조를 확인
	* Utensil.java 파일을 확인 후 Utensil, Dessert 모두 찾아냄
	* 이후 2번째 명령 인수인 Dessert.java를 처리할 때, 클래스 정의 중복을 발견
* `javac Main.java, javac Main.java Utensil.java` : "pancake" 출력
* `javac Dessert.java Main.java` : "potpie" 를 출력

#### 굳이 한 파일에 여러 클래스를 정의해야한다면?
* 정적 멤버 클래스를 사용하기

```java
public class Test {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }

  private static class Utensil {
    final String NAME = "pan";
  }

  private static class Dessert {
    final String NAME = "cake";
  }
}
```
