---
title: "web.xmlのバージョン別DTD・XSDの宣言方法"
date: 2015-04-07T23:24:07+09:00
tags: [Java, Maven] 
---
JavaでWebアプリを作る場合、Mavenのmaven-archetype-webappテンプレートを利用しています。
非常に便利なのですが、Servlet 3.1が出ている今、
生成されたwe.xmlがServlet 2.3の記述になっていてちょっと古すぎます。
そのため、毎回テンプレートから生成したあとにweb.xmlを書き換えなければなりません。
このちょっと面倒な作業を楽にするためにバージョン毎の宣言方法をメモしておきます。

<!-- MORE -->

## Servlet 2.3
maven-archetype-webappテンプレートで生成されるものです。

``` xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
 
<web-app>
</web-app>
```

## Servlet 2.4

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
         http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" >

</web-app>
```

## Servlet 2.5

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
           http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
           version="2.5">

</web-app>
```

## Servlet 3.0

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
           http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0">

</web-app>
```

## Servlet 3.1
現時点で最新のバージョンです。
このバージョンからネームスペースが`java.sun.com`から`xmlns.jcp.org`に変わるんですね。感慨深い。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
</web-app>
```
