---
title: "JavaFXでMarkdownエディタを作る"
date: 2014-12-06T06:42:41+09:00
tags: [Java, JavaFX] 
---

この記事は [JavaFX Advent Calendar 2014](http://www.adventar.org/calendars/380) の6日目です。  
昨日は[soutoku](http://www.adventar.org/users/5558)さんの[JavaFX:WYSIWYGエディタを作る](http://soutoku.hatenablog.com/entry/2014/12/05/013342)でした。  
明日は[@backpaper0](https://twitter.com/backpaper0)さんです。

JavaFX 楽しいですよね。

JavaFXには標準でWebページを表示するためのWebViewクラスがあり、これを使えばいろいろなことができます。
今回はこのWebViewクラスを使ってMarkdownエディタを作ってみることにします。
MarkdownといえばGitHubとかでも利用している人が多いかと思いますが、文書を記述するための軽量マークアップ言語です。
Markdownでテキストを入力し、それをパースしてWebViewに表示するという簡単な動作をするアプリケーションです。

<!-- MORE -->

## 環境

- OS: Mac OSX Yosemite 10.10
- メモリ: 4GB
- Java: Java SE Runtime Environment (build 1.8.0-b132)
- markdown4j-2.2-cj-1.0

Markdownのパースには[markdown4j](https://code.google.com/p/markdown4j/)を使うことにしました。

## 実際に動かしてみる

実際に動作している動画です。  

<iframe width="420" height="315" src="https://www.youtube.com/embed/zjFR_In-gik" frameborder="0" allowfullscreen></iframe>

## FXMLで外枠を作る

外枠を作るのはFXMLで書けばよいので簡単です。
IntelliJ IDEA 14 を使っているのでインラインScene Builderも使えますが...という状態なので
スタンドアロンのScene Builderを使いました。
ささっと作ったFXMLは以下のようになりました。

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.control.*?>
<?import javafx.scene.layout.*?>
<?import javafx.scene.text.TextFlow?>
<BorderPane maxHeight="-Infinity"
            maxWidth="-Infinity"
            minHeight="-Infinity"
            minWidth="-Infinity"
            prefHeight="400.0"
            prefWidth="600.0"
            xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1"
	    fx:controller="sample.Controller">
   <center>
      <SplitPane dividerPositions="0.5"
                 orientation="VERTICAL"
                 prefHeight="200.0"
                 prefWidth="160.0"
                 BorderPane.alignment="CENTER">
        <items>
          <AnchorPane minHeight="0.0" minWidth="0.0" prefHeight="100.0" prefWidth="160.0">
               <children>
                  <TextArea fx:id="textArea"
		            prefHeight="200.0"
                            prefWidth="200.0"
                            AnchorPane.bottomAnchor="0.0"
                            AnchorPane.leftAnchor="0.0"
                            AnchorPane.rightAnchor="0.0"
                            AnchorPane.topAnchor="0.0">
		  </TextArea>
               </children>
               <padding>
                  <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
               </padding>
            </AnchorPane>
          <AnchorPane minHeight="0.0" minWidth="0.0" prefHeight="100.0" prefWidth="160.0">
               <children>
                  <WebView fx:id="webView"
                           prefHeight="200.0" 
                           prefWidth="200.0" 
                           AnchorPane.bottomAnchor="0.0" 
                           AnchorPane.leftAnchor="0.0" 
                           AnchorPane.rightAnchor="0.0" 
                           AnchorPane.topAnchor="0.0" />
               </children>
               <padding>
                  <Insets bottom="5.0" left="5.0" right="5.0" top="5.0" />
               </padding>
            </AnchorPane>
        </items>
      </SplitPane>
   </center>
   <top>
      <MenuBar BorderPane.alignment="CENTER">
        <menus>
          <Menu mnemonicParsing="false" text="File">
            <items>
              <MenuItem mnemonicParsing="false" text="Close" />
            </items>
          </Menu>
          <Menu mnemonicParsing="false" text="Edit">
            <items>
              <MenuItem mnemonicParsing="false" text="Delete" />
            </items>
          </Menu>
          <Menu mnemonicParsing="false" text="Help">
            <items>
              <MenuItem mnemonicParsing="false" text="About" />
            </items>
          </Menu>
        </menus>
      </MenuBar>
   </top>
</BorderPane>
```

BorderPaneのtopに配置しているMenuBarは飾りです (^^;  
本体はBorderPaneのcenterにSplitPaneを配置し、上半分にTextArea、下半分にWebViewを表示します。

## コントローラを作る

この画面を操作するためのコントローラクラスを作ります。
コントローラがやることは、TextAreaに入力された値をパースしてWebViewに表示することです。
今回はTextAreaにイベントを張って、イベント発生毎にWebViewの内容を書き換える方法ではなく、
TextAreaのテキストプロパティにChangeListenerを設定し、値の変化を検知してWebViewを書き換えるようにします。
本当はWebView側にコンテンツをバインドできるプロパティが存在すればバインドを使いたいところです。

``` java
package sample;

import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.TextArea;
import javafx.scene.web.WebView;
import org.markdown4j.Markdown4jProcessor;

import java.io.IOException;
import java.net.URL;
import java.util.ResourceBundle;

public class Controller implements Initializable
{
    @FXML
    private TextArea textArea;

    @FXML
    private WebView webView;

    @Override
    public void initialize(URL location, ResourceBundle resources)
    {
        textArea.textProperty().addListener((observable, oldValue, newValue) -> {
            try {
                webView.getEngine().loadContent(new Markdown4jProcessor().process(newValue));
            }
            catch (StringIndexOutOfBoundsException | IOException e) {
                webView.getEngine().loadContent(newValue);
            }
        });
    }
}
```

ChangeListenerを設定するためには`TextArea#textProperty()`メソッドからStringPropertyを取得し、
そのプロパティの持つ`addListener`を使用します。
値の変更を検知するとChangeListenerのchangedメソッドがコールされるので、
ここでWebViewに対してHTMLを設定するようにします。
入力された文字列は`Markdown4jProcessor#process(String)`メソッドを経由してHTMLに変換されます。

`WebView#loadContent(String)`メソッドには`WebView#loadContent(String, String)`という引数を２つ取るメソッドも存在します。
引数が２つのほうは、第二引数にコンテンツタイプを指定できます。引数が１つのほうは内部で引数が２つメソッドを呼び出しいて、
第二引数には"text/html"を渡しています。ですので、デフォルトではHTMLを表示することになりますね。

## まとめとちょっと考察

これでシンプルなMarkdownエディタができました。
実は、ChangeListenerの代わりにInvalidationListenerを使うこともできます。
ChangeListenerとInvalidationListenerの違いは下記のサイトを参考にさせてもらいました。

- [JavaFX in the Box:JavaFX Hands on Lab](http://skrb.hatenablog.com/entry/2013/09/08/174826)
- [プログラムdeタマゴ:JavaFXのInvalidationListenerやChangeListenerやObservableListやBindingについて](http://d.hatena.ne.jp/nodamushi/20141012/1413136054)

``` java
textArea.textProperty().addListener(observable -> {
    final String value = ((StringProperty) observable).get();
    try {
        webView.getEngine().loadContent(new Markdown4jProcessor().process(value));
    }
    catch (StringIndexOutOfBoundsException | IOException e) {
        webView.getEngine().loadContent(value);
    }
});
```

ただ、InvalidationListenerを使うと次のような問題が発生します。

日本語入力においてChangeListenerだと確定前文字が入力された場合だけイベントが呼ばれるのですが、
InvalidationListenerだと確定前文字が入力された時と入力文字を確定した場合の2度イベントが発生します。
正確には、一回の値変更でイベントが2度発生するので計4回のイベントを受け取る形です。
確定前文字と入力文字を確定した時の文字は同じものであるため2回目のイベントが無駄になってしまいます。
使い方がまずいのかもしれないのですが、ChangeListenerを使ったほうが無難な様子です。

