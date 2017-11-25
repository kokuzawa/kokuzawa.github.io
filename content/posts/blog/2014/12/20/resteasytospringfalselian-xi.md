---
title: "RESTEasyとSpringの連携"
date: 2014-12-20T10:22:07+09:00
tags: [Java, JAX-RS, JavaEE] 
---

[Java EE Advent Calendar 2014](http://qiita.com/advent-calendar/2014/javaee)の20日の記事です。  
昨日は[@yoshioterada](https://twitter.com/yoshioterada)さんの「[Java EE 8 の新機能概要のご紹介](http://yoshio3.com/2014/12/19/java-ee-8-new-features/)」でした。  
明日は[@suke_masa](https://twitter.com/suke_masa)さんです。

Jersey-1.8を使ったアプリを最新の2.14に置き換えようと思ったところが始まりです。
サーバーがWildFlyだからRESTEasyが含まれているので、JerseyではなくRESTEasyを使えば良いのだけれども、
アプリ内でJersey MultiPartを使っているのでひとまずバージョンアップを試みたのですが、
いろいろ問題があって結局RESTEasyに置き換えました。

実際の運用はTomcatを使っているので、Tomcatでも動作する設定を考慮しています。
なので、WildFlyオンリーで考えた場合は不要な設定があるかもしれません。

<!-- MORE -->

## 環境

- OS: Mac OSX Yosemite 10.10
- Java: Java™ SE Runtime Environment (build 1.8.0-b132)
- メモリ: 4GB
- WildFly 8.0.0.Final

## pom.xml

下記のdiendencyが必要です。JettisonじゃなくてJacksonを使いたいのでそのdependencyも追加しています。
あとファイルアップロードも使いたいので、`resteasy-multipart-provider`も入れています。
`resteasy-spring`に依存してRESTEasyのコアライブラリは入るので定義の必要ありません。
Tomcatの場合、サーバにはJAX-RSの実装は入っていないのでscopeはcompileを指定します。
WildFlyの場合はscopeをcompileにすると起動時にエラーが発生するのでprovidedを指定します。
これは既にRESTEasyがサーバに含まれているから。

``` xml
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-servlet-initializer</artifactId>
    <version>3.0.10.Final</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-multipart-provider</artifactId>
    <version>3.0.10.Final</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-jackson-provider</artifactId>
    <version>3.0.10.Final</version>
</dependency>
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-spring</artifactId>
    <version>3.0.10.Final</version>
</dependency>
```

## web.xml

web.xml には下記を追加します。
url-pattarnが`/*`以外の場合は`resteasy.servlet.mapping.prefix`の設定が必要です。
`resteasy.scan`で自動的にJAX-RSのコンポーネントをスキャンする設定ができるのですが、
springと連携する場合は自動スキャンはしちゃダメ。
自動スキャンしようとすると`org.jboss.resteasy.plugins.spring.SpringContextLoaderListener`でエラーになります。

ということは、@Providerとか@PathがついたクラスはすべてSpringのコンポーネントにしておく必要があります。
あとApplicationのサブクラスはなくても大丈夫です。

`org.jboss.resteasy.plugins.spring.SpringContextLoaderListener`を追加しているので、
`org.springframework.web.context.ContextLoaderListener`は指定しちゃダメ。
指定すると起動に失敗します。

``` xml
  <context-param>
    <param-name>resteasy.servlet.mapping.prefix</param-name>
    <param-value>/rest</param-value>
  </context-param>
  
  <listener>
    <listener-class>org.jboss.resteasy.plugins.server.servlet.ResteasyBootstrap</listener-class>
  </listener>

  <listener>
    <listener-class>org.jboss.resteasy.plugins.spring.SpringContextLoaderListener</listener-class>
  </listener>
  
  <servlet>
    <servlet-name>Resteasy</servlet-name>
    <servlet-class>org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>Resteasy</servlet-name>
    <url-pattern>/rest/*</url-pattern>
  </servlet-mapping>
```

## まとめ

これでJAX-RSのリソースクラスにも@AutowiredでDIできるようになります。
RESTEasyのドキュメントに詳細に書いてあるんだけど、web.xmlの設定方法がServletのバージョンを考慮したパターンとか
いろいろありすぎて逆に困る。結局いろいろ試した末に上記の設定にたどり着きました。

同じようなことをしようとしている人の何かの参考になれば。

