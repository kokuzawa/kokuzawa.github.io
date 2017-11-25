---
title: "JavaFXのUIをJUnit形式でテストする"
date: 2015-12-10T10:10:10+09:00
tags: [Java, JavaFX] 
---
[Java Advent Calendar 2015](http://qiita.com/advent-calendar/2015/java)と
[JavaFX Advent Calendar 2015](http://qiita.com/advent-calendar/2015/javafx)の10日目の記事です。

昨日は下記のお二人でした。  

- Java Advent Calendarは[@namihira_k](https://twitter.com/namihira_k)さんの
「[パフォーマンスを意識したJavaコーディング](http://namihira.hatenablog.com/entry/20151209/1449667895)」
- JavaFX Advent Calendarは[@skrb](https://twitter.com/skrb)さんの
「[Interpolator で補間](http://skrb.hatenablog.com/entry/2015/12/09/212241)」

明日は下記のお二人です。

- Java Advent Calendarは[@bitter_fox](https://twitter.com/bitter_fox)さん
- JavaFX Advent Calendarは[@y_q1m](https://twitter.com/y_q1m)さん

<!-- MORE -->

## TestFXを知る

先月ダウンロードしたJava Magazine vol23に面白い記事が載っていました。
テストについて特集された中の、TestFXによるJavaFXのテストについての記事です。
TestFXはJavaFXのユーザ・インターフェースをJUnitベースでテストするためのAPIということで、
JUnitで書いたロジック通りにユーザ・インターフェースのテストが実施されます。
単純にロジックをなぞるだけではなく、実際にユーザ・インターフェースを操作した結果を判定してくれるようです。
これは、実際にテストを実行した際に、JavaFXのアプリ上でマウスカーソルが自動的に動いてボタンをクリックしたりすることからもわかります。

普段のプロジェクトでは、残念ながらJavaFXではなくFlexを使っているのですが、
ユーザ・インターフェース周りのテストの仕組みはあってもなかなか思ったようなテストができていないのが現実です。
TestFXはJUnitの延長上でテストができそうなので期待できそうです。

内容を説明する前に、実際に実行した際の動画を記録しました。
動画だと自動で動いているのかわからないと思いますが、
テスト起動後には何も操作をしていません。

<iframe width="420" height="315" src="https://www.youtube.com/embed/JdgX6ywJ4GY" frameborder="0" allowfullscreen></iframe>

## アプリの説明

テストに使ったアプリは、ラベルとボタンのあるシンプルなものです。
ボタンをクリックすることで、ラベルに「Hello World!」と表示します。

![](/images/post_image_40.png)

実際のコードは下記にあります。

[https://github.com/kokuzawa/javafx-test](https://github.com/kokuzawa/javafx-test)

## TestFXを設定

Mavenプロジェクトでは下記のDependencyを追加します。

```xml
<dependency>
    <groupId>org.loadui</groupId>
    <artifactId>testFx</artifactId>
    <version>3.1.2</version>
    <scope>test</scope>
</dependency>
```

## テストを書く

対象のテストクラスは、TestFXを使うために`org.loadui.testfx.GuiTest`クラスを継承します。
`org.loadui.testfx.GuiTest`クラスは`getRootNode()`メソッドを持ち、そのメソッドでテストしたい画面のFXMLをロードします。

```java
package org.katsumi;

import javafx.fxml.FXMLLoader;
import javafx.scene.Node;
import javafx.scene.Parent;
import org.junit.Test;
import org.loadui.testfx.GuiTest;

import java.io.IOException;
import java.util.logging.Level;
import java.util.logging.Logger;

import static org.junit.Assert.assertThat;
import static org.loadui.testfx.controls.Commons.hasText;

public class IndexControllerTest extends GuiTest
{
    @Override
    protected Parent getRootNode()
    {
        try {
            return FXMLLoader.load(getClass().getResource("index.fxml"));
        }
        catch (IOException e) {
            Logger.getLogger(IndexControllerTest.class.getName()).log(Level.SEVERE, "", e);
            return null;
        }
    }

    @Test
    public void testSay()
    {
        final Node node = find("#button");
        click(node);
        assertThat("#greeting", hasText("Hello World!"));
    }
}
```

テストメソッドは普通にJUnitの形式です。
内容ですが、まず`find("#button")`でfx:idがbuttonのコントロールを見つけます。
見つけたボタンコントロールを`click`メソッドを利用して実際にクリックします。
ラベルに「Hello World!」が設定されたことを`Assert.assertThat`で検証します。

## まとめ

TestFXはJUnitベースなので抵抗なくテストを実装することができました。
ただ、連続で何度か実行しているとエラーになることがありました。
原因を調べているのですが、まだちょっとわからない状態です。
とは言っても、エラーになるのは稀で、基本的は正常に動作します。

テストコードの導入はプロジェクトの最初の頃に決めておかないと、プロダクトコードがテストしにくい形で作られてしまうことが多々あります。
特にクライアント側のコードはその傾向が強いと思いますので、もしこれからJavaFXのプロジェクトを始める際のであれば、
TestFXの導入を検討してみてはいかがでしょうか。

今回この記事を書くきっかけになった、Java Magazineは下記からダウンロードすることができます。
[http://www.oracle.com/technetwork/jp/articles/java/overview/index.html?elq_mid=33486&sh=1612166126426151606143&cmid=JPFM15040092MPP006C005](http://www.oracle.com/technetwork/jp/articles/java/overview/index.html?elq_mid=33486&sh=1612166126426151606143&cmid=JPFM15040092MPP006C005)



