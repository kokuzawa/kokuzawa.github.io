---
title: "WildFly SwarmでEJBを試す"
date: 2015-11-22T22:33:53+09:00
tags: [Java, EJB, WildFly] 
---
WildFly Swarmを試すのも今日で３回目です。
だんだんと実装方法に慣れてきました。
この辺で当初の目的であったWildFly SwarmでEJBを使ってみたいと思います。
EJBが使えないのならSpring Bootで全然構わないわけで、
EJBが使えるかどうかはとても大事なところです。

<!-- MORE -->

## EJBを組み込む

EJBのモジュールを組み込みます。
よくよく考えてみると、EJBだけでは動きを確認するのが大変なので、
リクエストの受け口だけはJAXRSで作ります。
なので、JAXRSのモジュールも合わせて組み込みます。

また、JAXRSのリソースから`@Inject`でEJBをDIするにはWeldも必要です。
そのため、JAXRSのモジュールはwildfly-swarm-jaxrs-weldを利用することにします。

```xml
<dependencies>
    <dependency>
        <groupId>org.wildfly.swarm</groupId>
        <artifactId>wildfly-swarm-jaxrs-weld</artifactId>
        <version>1.0.0.Alpha5</version>
    </dependency>
    <dependency>
        <groupId>org.wildfly.swarm</groupId>
        <artifactId>wildfly-swarm-ejb</artifactId>
        <version>1.0.0.Alpha5</version>
    </dependency>
</dependencies>
```

## EJBを使ったアプリケーションを作る

まずはEJBです。

```java
package org.katsumi.ejb;

import javax.ejb.Stateless;

@Stateless
public class HelloEjb
{
    public String say()
    {
        return "Hello!";
    }
}
```

EJBを呼び出すRESTリソースです。

```java
package org.katsumi.ejb;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("/hello")
public class HelloResource
{
    @Inject
    private HelloEjb helloEjb;

    @GET
    public String hello()
    {
        return helloEjb.say();
    }
}
```

```java
package org.katsumi.ejb;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/rest")
public class MyApplication extends Application
{
}
```

## 実行する

今回はMainクラスを作るのではなく、Warファイルを生成し、
それを実行する形にします。
まずpom.xmlのpackagingをwarにします。

    <packaging>war</packaging>

さらにwarファイルを生成するので下記のプラグインをpom.xmlに追加します。

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>2.6</version>
        <configuration>
            <failOnMissingWebXml>false</failOnMissingWebXml>
        </configuration>
    </plugin>
    <plugin>
        <groupId>org.wildfly.swarm</groupId>
        <artifactId>wildfly-swarm-plugin</artifactId>
        <version>${version.wildfly-swarm}</version>
    </plugin>
</plugins>
```

準備ができたので下記のコマンドで実行します。

    mvn wildfly-swarm:run

## まとめ

WildFLy SwarmでEJBも問題なく呼び出せることがわかりました。
あとはJPAを使うことができれば、システム開発で使う一通りの機能が使えること確認できそうです。

