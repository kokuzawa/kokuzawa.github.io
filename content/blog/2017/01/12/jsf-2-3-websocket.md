---
title: "JSF 2.3 の Websocket を試す"
date: 2017-01-12T00:29:21+09:00
tags: [Java, JSF]
---
JSF 2.3 では新しい機能として Websocket が追加されます。  
JSF 2.3 はまだリリースされていませんが、先日 JSF 2.3-m09 が公開されたので、
これを使って Websocket を試してみようと思います。

<!-- more -->

## 環境

- macOS Sierra
- Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
- WildFly 10.1.0.Final
- Payara Server 4.1.1.164

## はじめに

今回やったことは下記に書いてあることそのままです。

[http://blog.payara.fish/jsf-2.3-the-websocket-quickstart-under-payara-server](http://blog.payara.fish/jsf-2.3-the-websocket-quickstart-under-payara-server)

ボタンをクリックすることでサーバサイドから時間を取得してそれを表示するアプリを作ります。
アプリの作り方は上記サイトを見てもらえばわかると思うので、ここでは上記のサイトには書かれていない、
JavaEE8に対応していない現状のAPサーバで JSF 2.3 を有効にする方法について説明します。

## WildFly 10.1.0.Final で試す

まず、普段利用している WildFly で動かそうとしてみました。
WildFly で JSF 2.3 を有効にするためには、WildFly が内包する JSF を無効にする必要があります。
無効にする方法は、下記の記述をした `WEB-INF/jboss-deployment-structure.xml` を用意します。

```xml
<?xml version="1.0"?>
<jboss-deployment-structure xmlns="urn:jboss:deployment-structure:1.2">

    <deployment>
        <exclude-subsystems>
            <subsystem name="jsf"/>
        </exclude-subsystems>
    </deployment>

</jboss-deployment-structure>
```

JSF 2.3 の jar ファイルは `WEB-INF/lib` フォルダに配置します。
この状態で WildFly にデプロイし、表示した画面でボタンをクリックすると下記のエラーが発生してしまいます。

```
Caused by: javax.faces.el.EvaluationException: javax.el.PropertyNotFoundException: /index.xhtml @15,73 action="#{pushBean.clockAction}": Target Unreachable, identifier 'pushBean' resolved to null
	at javax.faces.component.MethodBindingMethodExpressionAdapter.invoke(MethodBindingMethodExpressionAdapter.java:94)
	at com.sun.faces.application.ActionListenerImpl.processAction(ActionListenerImpl.java:102)
```

ApplicationScoped を付与した PushBean クラスがインスタンス化されていないようです。
スコープを変えたり、色々試してみたのですが動作は変わらないので諦めました。
動かす方法をご存知でしたら教えて頂けると嬉しいです。

## Payara Server で試す

参考にしたサイトは Payara を使っているので、次に Payara で試してみることにしました。
Payara も JSF を内包しているので何もせずにデプロイすると下記エラーが発生します。

```
Caused by: java.lang.reflect.MalformedParameterizedTypeException
	at sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl.validateConstructorArguments(ParameterizedTypeImpl.java:58)
	at sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl.<init>(ParameterizedTypeImpl.java:51)
	at sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl.make(ParameterizedTypeImpl.java:92)
	at sun.reflect.generics.factory.CoreReflectionFactory.makeParameterizedType(CoreReflectionFactory.java:105)
	at sun.reflect.generics.visitor.Reifier.visitClassTypeSignature(Reifier.java:140)
	at sun.reflect.generics.tree.ClassTypeSignature.accept(ClassTypeSignature.java:49)
	at sun.reflect.generics.visitor.Reifier.reifyTypeArguments(Reifier.java:68)
	at sun.reflect.generics.visitor.Reifier.visitClassTypeSignature(Reifier.java:138)
	at sun.reflect.generics.tree.ClassTypeSignature.accept(ClassTypeSignature.java:49)
	at sun.reflect.generics.repository.ClassRepository.getSuperclass(ClassRepository.java:90)
	at java.lang.Class.getGenericSuperclass(Class.java:777)
	at javax.enterprise.util.TypeLiteral.getTypeParameter(TypeLiteral.java:103)
	at javax.enterprise.util.TypeLiteral.getType(TypeLiteral.java:66)
	at com.sun.faces.cdi.CdiUtils.<clinit>(CdiUtils.java:76)
	... 65 more
```

Payara が内包する JSF を無効にする方法がわからなかったのですが、
Payara の場合は視点が違っていて、内包する JSF を無効にするのではなく、
Bundle した JSF を有効にする設定をしなければならないようです。
Bundle した JSF を有効にする方法は下記の記述をした `WEB-INF/glassfish-web.xml` を用意します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE glassfish-web-app PUBLIC
        "-//GlassFish.org//DTD GlassFish Application Server 3.1 Servlet 3.0//EN"
        "http://glassfish.org/dtds/glassfish-web-app_3_0-1.dtd">
<glassfish-web-app>
    <class-loader delegate="false" />
    <property name="useBundledJsf" value="true" />
</glassfish-web-app>
```

## まとめ

設定さえわかれば何も難しい話ではなかったんですが、
設定方法がわからなくて時間がかかってしまいました。
特に Payara の方は全然情報が見つからなくて苦戦...。
基本的にはリリースされてJavaEE8対応されたAPサーバで動かすことになるので知らなくても問題ないわけですが、
早期に試そうとすると色々とハードルがあり大変です。まあそれが楽しかったりするんですけどね。

## 参考

- [JSF 2.3 - The WebSocket Quickstart under Payara Server](http://blog.payara.fish/jsf-2.3-the-websocket-quickstart-under-payara-server)
- [How to update Mojarra version in GlassFish](http://stackoverflow.com/questions/10782528/how-to-update-mojarra-version-in-glassfish)

