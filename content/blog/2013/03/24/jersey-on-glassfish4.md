---
title: "jersey on glassfish-4"
date: 2013-03-24T12:32:00+09:00
tags: [Java, GlassFish] 
---

GlassFish-4を使ってみたらJerseyのパッケージが変わってた。
GlassFish-3.1.2.2までは以下のようにcom.sunパッケージにあるJerseyのサーブレットがロードできていたけど、

``` xml
<servlet-name>jersey-serlvet</servlet-name>
    <servlet-class>com.sun.jersey.spi.container.servlet.ServletContainer</servlet-class>
</servlet>
```

GlassFish-4ではcom.sunパッケージにJerseyはなく、org.glassfishのパッケージとなっている。

``` xml
<servlet-name>jersey-serlvet</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
</servlet>
```
