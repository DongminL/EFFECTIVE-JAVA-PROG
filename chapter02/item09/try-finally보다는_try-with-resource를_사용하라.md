자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많음.

자원을 닫는 것은 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어짐.

## try-finally
전통적으로 자원이 닫힘을 보장하는 수단

```java
static String firstLineOfFile(String path) throws IOException {  
    BufferedReader br = new BufferedReader(new FileReader(path));  
    try {  
        return br.readLine();  
    } finally {  
        br.close();  
    }  
}

/////

static void copy(String src, String dst) throws IOException {  
    InputStream in = new FileInputStream(src);  
    try {  
        OutputStream out = new FileOutputStream(dst);  
        try {  
            byte[] buf = new byte[1024];  
            int len;  
            while ((len = in.read(buf)) > 0) {  
                out.write(buf, 0, len);  
            }  
        } finally {  
            out.close();  
        }  
    } finally {  
        in.close();  
    }  
}
```

`copy` 메서드의 경우 close가 두번 호출하게 되어 코드가 상당히 더러워짐.

## try-with-resource
`try-with-resource` 를 사용하려면 AutoCloseable 인터페이스를 구현해야 합니다.

```java
static String firstLineOfFile(String path) throws IOException {  
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {  
        return br.readLine();  
    }
}

/////

static void copy(String src, String dst) throws IOException {  
    try (InputStream in = new FileInputStream(src);  
         OutputStream out = new FileOutputStream(dst)) {  
        byte[] buf = new byte[1024];  
        int len;  
        while ((len = in.read(buf)) > 0) {  
            out.write(buf, 0, len);  
        }  
    }
}
```

AutoCloseable를 구현하면 훨씬 가독성이 좋고 문제를 진단하기도 좋습니다.

여러 개의 예외가 발생했을 때 프로그래머에게 보여줄 예외 하나만 보존되고 여러 개의 다른 예외가 숨겨질 수도 있는데, 숨겨진 예외들도 버려지지 않고 스택 추적 내역에 'suppressed' 라는 꼬리표를 달고 출력됩니다.

```java
static class TestResource implements Closeable {  
    @Override  
    public void close() throws IOException {  
        throw new IOException("리소스 닫는 중 예외 발생!");  
    }  
}  
  
public static void main(String[] args) {  
    try (TestResource resource = new TestResource()) {  
        throw new RuntimeException("기본 예외 발생!");  
    } catch (Exception e) {  
        System.out.println("기본 예외: " + e.getMessage());  
  
        // 억제된 예외 출력  
        for (Throwable suppressed : e.getSuppressed()) {  
            System.out.println("억제된 예외: " + suppressed.getMessage());  
        }  
    }  
}
/* 출력
기본 예외: 기본 예외 발생!
억제된 예외: 리소스 닫는 중 예외 발생!
*
```

또 보통의 `try-finally` 처럼 `catch`절을 쓸 수 있습니다.  
catch절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외 처리 가능

>`try-finally`로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, `try-with-resource`로는 정확하고 쉽게 자원을 회수할 수 있다.
