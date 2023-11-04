# item9) try-with-resources를 사용하자

* 직접 close 메서드를 호출해 닫아줘야하는 자원이 많지만 클라이언트의 실수로 예측할 수 없는 성능 문제가 발생할 수 있다.
* 안전망으로 finalizer를 활용하지만 반드시 닫아줄 것이라는 보장이 없다.

## **try-finally 구조**

* 예외는 try 블록과 finally 블록 모두에서 발생 가능하다.
* 두 블록 모두에서 예외가 발생하면, finally 블록에서의 예외에 대한 정보는 숨겨져 디버깅이 어려울 수 있다.
* catch절을 함께 사용해 다수의 예외 처리 가능

## **try-with-resources 구조**

* AutoCloseable을 구현한 클래스만 사용 가능하다. `void close()` 메서드를 구현하면 된다.
* 짧고 가독성이 좋으며 만들어지는 예외 정보가 유용하므로 문제 진단하기가 좋다.
* 숨겨진 예외들이 버려지지 않고 스택 추적 내역에 숨김 표시를 달고 출력된다.
* getSuppressed 메서드 사용 시 숨겨진 예외 가져오기 가능
* catch절을 함께 사용해 다수의 예외 처리 가능

```java
static String firstLineOfFile(String path, String defaultVal) {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	}
	catch (IOException e) {
		return defaultVal;
	}
}
```
