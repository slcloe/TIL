# Item 16 - public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하기

``` java
// 부적적한 코드 - 캡슐화를 지키지 않은 코드
public class Point {
  public int x; 
  public int y;
}
```
#### 캡슐화의 이점을 살려 프로그래밍 하기!

1. `가변`필드들은 모두 private 로 바꾸기
2. public 접근자 getter 만들기

``` java
public class Point {
  public int x;
  public int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public int getX() { return x; }
  public int getY() { return y; }

  public void setX(int x) { this.x = x; }
  public void setY(int y) { this.y = y; }
}

```
* package-private / private 중첩 클래스라면 데이터 필드를 노출해도 상관없다.
* 가변 객체를 final로 선언해도 객체 내용이 변경될 수 있다.
