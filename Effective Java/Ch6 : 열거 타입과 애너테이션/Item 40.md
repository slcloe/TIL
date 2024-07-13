# Item 40 - @Override 애너테이션을 일관되게 사용하기

#### @Override 를 작성하지 않았다면?
```java
public boolean equals(Bigram b) {
	return b.first == first && b.second == second; 
}
``` 
* 의도는 Override일지 몰라도, 이 경우는 Override를 한게 아닌 Overloading을 한 코드이다.
* 이 경우 equals가 올바르게 작동하지 않아 HashMap, HashSet 구현 시 문제가 될 수 있다.
* 심지어, 컴파일, 런타임 에러가 나지 않아 잘못 작동함에도 오류의 원인을 찾기 힘들어진다.

```java
@Override public boolean equals(Bigram b) {
	return b.first == first && b.second == second; 
}
``` 
* 이 경우는 Override를 작성했기 때문에 컴파일러가 미리 어떤 부분에서 오류가 났는지를 알려주게 된다.

=> 재정의한 모든 메서드에 @Override를 달아 실수를 방지하자.!
<!-- 
```java

``` 
-->