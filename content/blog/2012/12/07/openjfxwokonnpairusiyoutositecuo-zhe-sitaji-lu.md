---
title: "OpenJFXをコンパイルしようとして挫折した記録"
date: 2012-12-07T21:05:00+09:00
tags: [Java, JavaFX]
---

[JavaFX Advent Calendar 2012](http://atnd.org/events/33874)の7日目のエントリーです。  
昨日は[@fukai_yas](https://twitter.com/fukai_yas)さんの「[AppletでFXMLを使って罠にハマる](http://blog.livedoor.jp/fukai_yas/archives/20772650.html)」です。  
明日は[@btnrouge](https://twitter.com/btnrouge)さんです。

## JOptionPaneが使いたい
JavaFX 2.2.3で業務に利用する簡単なツールを作っていたのですが、入力エラーを通知する為に、
SwingでいうところのJOptionPane相当のものを探したのですがみつかりませんでした。
誰か知っている人はいないだろうかとTwitterでつぶやいてみたところ、@skrbさんよりこんなお返事を頂きました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr"><a href="https://twitter.com/kokuzawa">@kokuzawa</a> Project Sandboxというのがあって、そこでJOptionPaneのように使えるDialogクラスを提供してますよ。正式にはJavaFX 8で入る予定です。 <a href="http://t.co/w6AswYKA">http://t.co/w6AswYKA</a></p>&mdash; Yuichi Sakuraba (@skrb) <a href="https://twitter.com/skrb/status/274416797602705408">2012年11月30日</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Project SandboxでJOptionPane相当のDialogsクラスが提供されているということ。JavaFX 8 に入るらしい。
すばらしい！すばらしいけど、今使いたい。そこでひとまずProject Sandboxを動かしてみる事にしました。

## OpenJFXをビルドする
Project SandboxはOpenJFXのSandboxプロジェクトで、これを利用する為には、Mercurialに登録されているソースをコンパイルする必要があるようです。
そこで[OpenJFXのページ](http://openjdk.java.net/projects/openjfx/getting-started.html)に書かれているビルド手順を実行します。
でもこれ、記述が古い。JDKのフォルダ構成とか、今のものとはかなり違う。さて困った。

そこで新たな情報を求めていると、[Building OpenJFX](https://wikis.oracle.com/display/OpenJDK/Building+OpenJFX)という、
まさに望んだ通りのページがありました。手順は次のようになります。
また、コンパイルにはJDK 8 が必要です。

* ``mkdir -p ~/open-jfx``
* ``cd ~/open-jfx``
* ``hg clone http://hg.openjdk.java.net/openjfx/8/master``
* ``cd master``
* ``mkdir -p artifacts/sdk/rt/lib``
* ``cp -r <PATH TO JDK>/jre/lib/jfxrt.jar artifacts/sdk/rt/lib``
* ``hg clone http://hg.openjdk.java.net/openjfx/8/master/rt``
* ``cd rt``
* ``ant``

今度こそ、コンパイル出来そうな感じです。最後のantを実行すると、何個かのモジュールのjarが出来上がっていきます。
が、javafx-ui-commonというモジュールのコンパイルが通りません。JDKのバージョンの問題かもしれないと思ったのですが、
JDK 8 はJDK 8 b65というもので、この時点では最新のものです。
ここで1日試行錯誤を繰り返したものの、解決の糸口はなく、ビルドはあきらめる事にしました。

## やっぱりJOptionPaneが使いたい
OpenJFXのコンパイルにかなりの時間を取られたのですが、本来の目的はJOptionPane相当のものが使いたいだけです。
最初に教えて頂いた[ページ](http://fxexperience.com/2012/10/announcing-the-javafx-ui-controls-sandbox/)のコメント欄を読んでいると、
Marco Jakobさんという方がJavaFX 2.2でも利用できるDialogsがあると書いています。やったね、これじゃん！

ということで、早速[リンク先](http://edu.makery.ch/blog/2012/10/30/javafx-2-dialogs/)に行ってみると、
使い方が大変詳しく記述されています。詳しくはリンク先をご覧頂くとして、例えば下記のようなコードを書くだけで、
ダイアログが表示されます。

```
Dialogs.showErrorDialog(stage, null, "名前が入力されていません。", "Error");
```

## まとめ
JavaFX 8 になると、これ以外にも業務で利用できそうなTreeTableViewなど、様々なもの増えるようです。
JDK 8 が待ち遠しいですが、現状ではコンパイルもままならないので、今回さわったDialogsクラスをしばらくメインで使わせてもらうつもりです。
しかし、JavaFXはUIが綺麗で楽しくなりますね。

