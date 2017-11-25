---
title: "IntelliJ IDEAでglassfish-web.xmlを自動生成するには"
date: 2013-03-21T00:04:00+09:00
tags: ["IntelliJ IDEA"]
---

NetBeansだとsun-web.xmlとして自動生成されるファイルのことです。
JavaEE6の<a href="http://docs.oracle.com/javaee/6/tutorial/doc/javaeetutorial6.pdf" target="_blank">ドキュメント</a>を読む限り、
glassfish-web.xmlが正式名称のようですが...。

このファイル、通常は必要ないのですが、GlassFishのBASIC認証とかDIGEST認証にJDBCレルムを使おうとすると必要になります。
XMLファイルなので、DTDのパスとか...まあその辺の入力が面倒な訳で、テンプレートは検索すれば見つかりますけど、出来ればIntelliJで生成できないかなと。

## 手順
画面上部のツールバーからProject Structureを選択。もしくはCmd+;

![](/images/post_image_6.png)

表示された画面からModules->Webと選択し、右ペイン中にあるAdd Aplication Server specific descriptorボタンをクリック。

![](/images/post_image_7.png)

表示されたポップアップのApplication ServerをGlassfish Serverに、Descriptorは選択肢が2つで最後にOracleとなっている方を選択。
(以下のイメージでは既にOracleを設定済みなのでSunしか表示されていない。)

![](/images/post_image_8.png)

これでglassfish-web.xmlのテンプレートが自動生成される。

**glassfish-web.xml:**

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE glassfish-web-app PUBLIC "-//GlassFish.org//DTD GlassFish Application Server 3.1 Servlet 3.0//EN"
    "http://glassfish.org/dtds/glassfish-web-app_3_0-1.dtd">
<glassfish-web-app>
</glassfish-web-app>
```

あとはここに必要な記述を追加して行くだけ。IntelliJは補完が強力なので、ここまでファイルが出来てくれれば後は楽。

