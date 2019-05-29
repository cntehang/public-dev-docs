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

## 时区概念

程序中对时间处理，是根据服务器本地时间来的，所以对时间处理（转换，比较），必须要有时区的概念

反例：
```java
public static boolean isDateTimeGreaterThanNowOfBeijing(String dateTimeStr) {
  DateTime dateTime = DateTime.parse(dateTimeStr, DATE_TIME_PATTERN); // 转换时未指定时区，下面的比对会错误
  DateTime now = DateTime.now(DateTimeZone.forID(ZONE_SHANGHAI));
  return dateTime.getMillis() > now.getMillis();
}
```

正例：
```java
public static DateTime getCstNow() {
  return new DateTime(DateTimeZone.forID(ZONE_SHANGHAI)); // 指定时区
}
```