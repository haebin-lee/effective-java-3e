# item09. try-finally 보다는 try-with-resources를 사용하라. 

자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다. `InputStream`, `OutputStream`, `java.sql.Connection` 등이 좋은 예다. 

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally`가 쓰였다. 예외가 발생하거나 메서드를 반환되는 경우를 포함해서 말이다. 

try-finally - 더이상 자원을 회수하는 최선의 방책이 아니다!
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

자원이 둘 이상이면 try-finally 방식은 너무 지저분하다.
```java
 static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n; 
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
예외는 try블록과 finally 블록 모두에서 발생할 수 있는데 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다. 

하지만 이런 문제들은 자바7의 `try-with-resources` 덕에 모두 해결되었다. 

try-with-resources 자원을 회수하는 최선책
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    }
}
```

복수의 자원을 처리하는 try-with-resources - 짧고 매혹적이다.
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n; 
        while ((n=in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

`try-with-resources` 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기 훨씬 좋다. 

try-with-resources를 catch절과 함께 쓰는 모습 
```java
static String firstLineOfFile(String path, String defaultValue) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))){
        return br.readLine();
    } catch (IOException e) {
        return defaultValue;
    }
}
```

```
핵심 정리 

꼭 회수해야하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다. 
```