---
title: "WildFly SwarmでJAXRSを試す"
date: 2015-11-14T16:08:44+09:00
tags: [Java, WildFly, JAXRS] 
---
WildFly Swarmのサイトにも記載されていますが、WildFly Swarmは自己完結型のJava microservicesを作成するのに役立つプロジェクトとのこと。
この分野だとSpring Bootの方が運用実績もあり、先行しているようですが、
将来的にはEJBも使いたいのでWildFly Swarmの方を使ってみます。
WildFly Swarmは一つのモジュールというわけではなく、JavaEEの仕様毎に複数のモジュールに分かれていて、
自分の必要なモジュールを取り込んで利用する形になるようです。

<!-- MORE -->

## JAXRSを組み込む

今回は数あるモジュールの中からJAXRSのモジュールを利用してみます。
2015年11月時点での最新バージョンは1.0.0.Alpha5です。
pom.xmlに下記のdependencyを追加します。

```xml
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>wildfly-swarm-jaxrs</artifactId>
    <version>1.0.0.Alpha5</version>
<dependency>
```

これを依存グラフで見てみると...依存がすごいです（笑）

![](/images/post_image_39.png)

JAXRSのモジュールを組み込んだだけではビルドしても実行できないので、
下記のpluginもpom.xmlに追加します。

```xml
<plugin>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>wildfly-swarm-plugin</artifactId>
    <version>1.0.0.Alpha5</version>
    <executions>
        <execution>
            <goals>
                <goal>package</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## JAXRSアプリケーションを作る

WildFly Swarmの設定が一通り終わったので、次はJAXRSアプリケーションを作ります。
特別なことはなく、普通のJAXRSアプリケーションです。

```java
package org.katsumi;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/rest")
public class MyApplication extends Application
{
}
```

```java
package org.katsumi;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("/hello")
public class HelloResource
{
    @GET
    public String hello()
    {
        return "Hello World!";
    }
}
```

## どうやってうごかすの？

JAXRSアプリケーションも作ったけれどどうやって動かすのか？  
WildFly Swarmでは通常のJavaアプリケーションのようにmainメソッドから動かします。
そのため、mainメソッドを持つクラスを新たに作成します。

```java
package org.katsumi;

import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.wildfly.swarm.container.Container;
import org.wildfly.swarm.jaxrs.JAXRSArchive;

public class Main
{
    public static void main(String... args) throws Exception
    {
        // コンテナの生成
        // Archiveを生成する前にインスタンス化しておかないと実行時にエラーが発生
        final Container container = new Container();

        // ShrinkWrapで仮想アーカイブを作成
        final JAXRSArchive archive = ShrinkWrap.create(JAXRSArchive.class);
        archive.addClass(MyApplication.class);
        archive.addClass(HelloResource.class);
        archive.addAllDependencies();

        container.start().deploy(archive);
    }
}
```

mainメソッド内では、ShrinkWrapを利用して生成した仮想アーカイブを起動したコンテナにデプロイします。
コード中のコメントにも書きましたが、アーカイブを作る前にコンテナをインスタンス化しておかないと、
実行時にエラーになります。これで半日悩んだ..orz

で、ここで作ったMainクラスをswarm-pluginに教える必要があります。
設定を追加したwildfly-swarm-pluginが下記になります。

```xml
<plugin>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>wildfly-swarm-plugin</artifactId>
    <version>1.0.0.Alpha5</version>
    <configuration>
        <mainClass>org.katsumi.Main</mainClass>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>package</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Mavenでビルド後に下記の方法で実行することができます。

### IDEの場合

Mainクラスを実行

### Mavenを利用する場合

    mvn wildfly-swarm:run

### Jarファイルを実行する場合

    jar -jar target/projectname-swarm.jar

## 余談

今回のアプリですが、IntelliJ IDEA 15で作っています。
インストールしたままの環境でMavenビルドをしたのですが、下記エラーが発生してビルドができない状態でした。

    java.lang.NoClassDefFoundError: org/eclipse/aether/RepositorySystemSession

原因はMavenのバージョンが古いためで、IntellijにデフォルトでバンドルされているMavenのバージョンは3.0.5であり、
このバージョンではエラーが発生するので、別途バージョン3.2.5をインストールしてそれを参照するようにしました。

