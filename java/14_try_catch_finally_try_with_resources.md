## try-catch-finally vs try-with-resources
[참고 링크](https://mangkyu.tistory.com/217)
### try-catch-finally

```java
public static void main(String[] args) {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("file.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        // catch
    } finally {
        // close
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}
```
* Java7 이전 일반적으로 사용했던 방식
* 사용 후 반납해주어야 하는 자원들은 Closable 인터페이스를 구현하고 있으며 사용 후에 close 메서드들을 호출했어야 함
* try 및 finally 작업에서 예외가 발생할 경우 스택 트레이스는 finally에서의 예외만 보여줌
### try-with-resources
```java
try (FileReader fr = new FileReader("file.txt")) {
    int i;
    while ((i = fr.read()) != -1) {
        System.out.println((char) i);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```
* Java7 이후 등장
* 기존 try-catch-finally의 문제: 자원 반납에 의한 번거로운 코드, 실수/에러로 자원을 반납하지 못하는 경우 발생
* AutoClosable 인터페이스를 구현하고 있는 자원에 대해 try-with-resources를 적용하도록 함
* finally를 작성하지 않아도 된다.
* 기존 Closable이 AutoClosable을 상속하도록 설정 - `하위 호환성 100% 달성`을 위함
* try 및 finally 작업에서 예외가 발생할 경우 모든 발생한 예외를 보여줌
* 여러 finally 작업을 할 때 한 자원에서 예외가 발생할 경우 다른 자원의 close를 제대로 하지 못했던 문제를 해결해 줌 - 컴파일러가 finally에서의 모든 경우를 try-catch-finally로 변환
