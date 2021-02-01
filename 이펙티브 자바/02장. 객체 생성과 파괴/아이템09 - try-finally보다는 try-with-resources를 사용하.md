# try-finally보다는 try-with-resources를 사용하라

- 자바 라이브러리에는 clsoe 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

  > `InputStream`, `OutputStream`, `java.sql.Connections` 등이 좋은 예이다.

- 자원 닫기는 클라이언트가 놓치기 쉬워, 예측할 수 없는 성능 문제로 이어지기도 한다.
- 이런 자원 중 상당수가 안전망으로 finlizer를 활용하고 있지만 finlizer는 그리 믿을만하지 못하다.(아이템8)

## try-finally는 더이상 자원을 회수하는 최선의 방책이 아니다.

- try-finally 방식은 코드를 지저분하게 한다.
- 자원이 하나일 때면 봐줄만 하지만 두 개 이상이 될 경우 너무 지저분해진다.

## try-with-resources

- 이 구조를 사용하려면 `AutoCloseable` 인터페이스를 구현해야한다.
- 자바의 수많은 인터페이스와 클래스는 이미 구현해 두었다.
- 닫아야할 자원을 뜻하는 클래스를 작성한다면 `AutoCloseable` 인터페이스를 반드시 구현하자.

### 예시

1. 닫아야할 자원이 **하나**일 경우

   ```java
   static String firstLineOfFile(String path) throw IOException {
   	try (BufferedREader br = new BufferedReader(new FileReader(path))) {
   		return br.readLine();
   	}
   }
   ```

2. 닫아야할 자원이 **복수**일 경우

   ```java
   static void copy(String src, String dst) throws IOException {
   	try (InputStream in = new FileInputStream(src);
   		OutputStream out = new FileOutputStream(dst)) {
   		byte[] buf = new byte[BUFFER_SIZE];
   		int n;
   		while ((n = in.read(buf)) >= 0)
   			out.wirte(buf, 0, n);
   	}
   }
   ```

- try-with-resources는 짧고 읽기 수월하다.
- 문제를 진단하기도 더욱 좋다.
