자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.

자원이 둘 이상이면 코드가 지저분 해진다.
또한 예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데 예외가 생긴 경우 중 try 블록 안에서 생긴 예외가 close 메소드에도 영향을 끼치는 경우 close 메소드의 예외가 try 블록 안에서 생긴 예외를 먹어버려 디버깅이 어렵게 된다.

이런 문제를 해결하기 위해 try-with-resources 가 등장 하였다.
이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야한다.

```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

코드를 이렇게 try-with-resources로 고치면 close를 알아서 호출해주며,
close 호출에서 예외가 발생했을 때, close에서 발생한 예외는 숨겨지고 첫 번째 예외가 기록된다.
이렇게 실전에서는 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질 수 있다.

이렇게 숨겨진 예외들은 스택 추적 내역에 suppressed 꼬리표를 달고 출력된다.
또한 자바7에서 Throwable에 추가된 getSuppressed 메서드를 쓰면 프로그램 코드에서 가져올 수 있다.
