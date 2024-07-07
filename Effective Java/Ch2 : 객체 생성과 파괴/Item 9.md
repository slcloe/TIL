# Item 9 - try-finally보다는 try-with-resources를 사용하기

> **InputStream, java.sql.Connection처럼 자원을 닫아줘야하는 라이브러리가 존재한다.**
> 하지만 클라이언트가 놓친다면 성능문제까지 이어질 수 있다.

##### try-finally의 문제점
* 닫아야 할 자원이 2개 이상이라면 코드가 더러워진다.
* 중첩 구문으로 사용하게 되면 스택 추적 내역에 n번째 중 첫 번째 예외에 관한 정보를 찾을 수 없다.

##### try-with-resources 사용하기
* AutoCloseable 인터페이스를 구현해야한다.
```java
static void copy(String src, String dst) throws IOException { 
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) { 
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n); }
}
```
* 예외 추적 시 한개의 예외를 보여주고 나머지는 Supperessed 로 출력된다.
-> 이는 Throwable의 getSuppressed 메서드를 이용하면 코드에서 가져올 수 있다. `Java 7`


