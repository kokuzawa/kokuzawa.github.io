---
title: "WildFly SwarmでJSFを試す"
date: 2015-11-16T22:58:09+09:00
tags: [Java, JSF, WildFly]
---
昨日はWildFly SwarmでJAXRSを触ったので、今日はJSFを試してみることにします。  

<!-- MORE -->

## JSFを組み込む

仕様毎にモジュールが分かれているので、JAXRSの時と同じく、今回はJSFのモジュールを取り込みます。
あと、ここで特に記載はしませんがwildfly-swarm-pluginももちろん設定する必要があります。

```xml
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>wildfly-swarm-jsf</artifactId>
    <version>1.0.0.Alpha5</version>
</dependency>
```

## JSFアプリケーションを作る

JSFアプリケーションを作ると言ってもJavaのコードを書くわけではなく、
動くことが分かれば良いのでXHTMLファイルだけを作るだけにします。

```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">

<body>
    <h:outputText value="Hello JSF!"/>
</body>

</html>
```

あと、JAXRSの時と違い、web.xmlを作る必要があります。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <context-param>
        <param-name>javax.faces.PROJECT_STAGE</param-name>
        <param-value>Development</param-value>
    </context-param>
    
    <servlet>
        <servlet-name>Faces Servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>
</web-app>
```

## 動かす

JAXRSの時と同じく、mainメソッドから実行するため、mainメソッドを持つクラスを作ります。
今回はJAXRSArchiveではなく、WARArchieをデプロイします。

```java
package org.katsumi.jsf;

import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.ClassLoaderAsset;
import org.wildfly.swarm.container.Container;
import org.wildfly.swarm.undertow.WARArchive;

public class Main
{
    public static void main(String... args) throws Exception
    {
        final Container container = new Container();

        final WARArchive warArchive = ShrinkWrap.create(WARArchive.class);
        warArchive.addAsWebResource(
                new ClassLoaderAsset("index.xhtml", Main.class.getClassLoader()), "index.xhtml");
        warArchive.addAsWebInfResource(
                new ClassLoaderAsset("WEB-INF/web.xml", Main.class.getClassLoader()), "web.xml");
        warArchive.addAllDependencies();

        container.start().deploy(warArchive);
    }
}
```

実行方法は下記の３通りです。

- IDEでmainメソッドを実行
- mvn wildfly-swarm:run
- jar -jar target/projectname-swarm.jar

これで実際に実行しようとすると下記のエラーが発生します。

```java
at org.wildfly.extension.undertow.deployment.UndertowDeploymentService$1.run(UndertowDeploymentService.java:85)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
	at org.jboss.threads.JBossThread.run(JBossThread.java:320)
```

これは`wildfly-swarm-weld`を取り込んでいないためなので、
下記の記述をpom.xmlに追加します。

```xml
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>wildfly-swarm-weld</artifactId>
    <version></version>
</dependency>
```

## まとめ

ドキュメントを読んだだけではWeldが必要には見えなくて、
結局サンプルコードと何遍も見比べる必要がありましたが、
動いてしまえば、後は非常に快適です。
次はEJBが使えるのか試してみないと。

    
