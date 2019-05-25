# Java 代码最佳实践

## try with resource

尽量使用 Java 7 和 Java 9 带来的新语法减缓代码。注意，当有多个资源时，各个资源用`;`分开，都应该都做 try 后面的括号里面。

```java
// use the folloiwng in Java 7 and Java 8
try (InputStream stream = new MyInputStream(...)){
    // ... use stream
} catch(IOException e) {
   // handle exception
}

// use the following in Java 9 and later
InputStream stream = new MyInputStream(...)
try (stream) {
    // ... use stream
} catch(IOException e) {
   // handle exception
}

// NOT the following
InputStream stream = new MyInputStream(...);
try {
    // ... use stream
} catch(IOException e) {
   // handle exception
} finally {
    try {
        if(stream != null) {
            stream.close();
        }
    } catch(IOException e) {
        // handle yet another possible exception
    }
}
```
