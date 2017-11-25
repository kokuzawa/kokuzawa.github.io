---
title: "JSF2.0でCSSリソース内の画像を読み込む"
date: 2013-06-10T03:10:00+09:00
tags: [Java, JSF] 
---
JSF 2.0 でCSSリソース内の画像を読み込むにはコツが必要です。  
Bootstrapのglyphicons-halflings.pngを読み込むのに暫く悩んだのでそのメモなど。

<!-- MORE -->

フォルダ構成は次のようになっています。（Mavenです）

```
root
 +-- src
     +-- webapp
         +-- resources
             +-- css
             |   +-- bootstrap.min.css
             +-- img
                 +-- glyphicons-halflings.png
```

JSFなのでXHTMLを使い、CSSの読み込みは以下のようにします。

``` xml
<h:outputStylesheet library="css" name="bootstrap.min.css"/>
```

このとき、bootstrap.min.css内のイメージのパスは`../img/glyphicons-halflings.png`となっていますが、
このままでは画像が読み込まれません。まあ、パスからも明らかですね。

問題を解決するには次のように、イメージのパスを変更する必要があります。

``` css
background-image:url("#{resource['img/glyphicons-halflings.png']}")
```

## Jun 20, 2013 追記
twitterで[@den2sn](https://twitter.com/den2sn)さんに教えてもらったのですが、outputStylesheetを下記のようにlibraryを削除して宣言することにより、
bootstrap.min.cssを書き換えなくても良くなります。

``` xml
<h:outputStylesheet name="css/bootstrap.min.css"/>
```

