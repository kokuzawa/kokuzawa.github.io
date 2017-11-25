---
title: "java.sql.Timestamp の振る舞い"
date: 2016-06-27T23:41:21+09:00
tags: [Java]
---
Java6 と Java8 で振る舞いが変わっていたのでメモ。

Java6 では下記コードがエラーにならず結果が出力されます。

```java
System.out.println(java.sql.Timestamp.valueOf("2016-13-01")); // 2017-01-01
```

ところが Java8 だと下記のエラーが発生します。

```java
Exception in thread "main" java.lang.IllegalArgumentException: Timestamp format must be yyyy-mm-dd hh:mm:ss[.fffffffff]
	at java.sql.Timestamp.valueOf(Timestamp.java:204)
```

もちろん存在しない日付、例えば 2016-12-32 などを指定した場合にもエラーとなります。
Java6 から Java7 になる際に `java.sql.Timestamp` に対してかなりの数のバグフィックスが行われたようで、
おそらくその修正のどこかで振る舞いが変わったのだと思います。

