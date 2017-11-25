---
title: "IntelliJ IDEA 13で作るJavaFXアプリケーション"
date: 2013-12-08T00:15:00+09:00
tags: [JavaFX, Java] 
---
この記事は、[JavaFX Advent Calendar 2013](http://www.adventar.org/calendars/146)の8日目です。  
昨日は[@aoetk](http://twitter.com/aoetk)さんの[ListViewやTableViewのセルをカスタマイズする方法](http://d.hatena.ne.jp/aoe-tk/20131206/1386345344)でした。  
明日は[@sk44_](http://twitter.com/sk44_)さんです。

## 祝！IntelliJ IDEA 13 リリース
IntelliJ IDEA 13 が 12月3日にリリースされました！めでたいですね！  
たぶん説明はいらないと思うので書きませんが、Java界隈の人なら知る人ぞ知る素敵なIDEです。  
IntelliJの最新版であるver.13はJavaFXもサポートされているので、
今回はこれを使ってJavaFXの簡単なアプリケーションを作ってみたいと思います。  
6日目の記事で[@khasunuma](http://twitter.com/khasunuma)さんが書かれた、
「[e(fx)clipseで作るJavaFXアプリケーション](http://www.coppermine.jp/docs/programming/2013/12/efxclipse.html)」のIntelliJ版ですね。

## 早速作ってみる
IntelliJを初めて起動すると以下のような画面が表示されるので、右側にある「Create New project」を選択します。

![](/images/post_image_16.png)

今度はプロジェクトの作成画面が表示されるので、
左側からプロジェクトのタイプとして「JavaFX Application」を選択、右側の「Project name」に任意のプロジェクト名を入れます。
今回は「HelloJavaFX」という名前にしました。

![](/images/post_image_17.png)

Finishボタンをクリックすると、プロジェクトが作られます。
プロジェクトには簡単なファイルが3つ用意されていて、それぞれ以下のような内容になっています。

![](/images/post_image_18.png)

![](/images/post_image_19.png)

![](/images/post_image_20.png)

Mainクラスがちゃんと書かれているのでこれだけでも動きます。
初めてJavaFXのアプリケーションを作る人に必要最低限のファイルとコードを示すのに十分です。
これ以上のコードがあると、本来は不必要なものまで必要だと勘違いしてしまうと思います。
ただControllerとFXMLは空っぽなので、このままだと枠しか表示されなくて面白くない...。  
ボタンをクリックしたら文字を表示するというぐらいはやりたいので、
ボタンとラベルをFXMLに追加していきます。

``` xml
<?import javafx.geometry.Insets?>
<?import javafx.scene.layout.GridPane?>

<?import javafx.scene.control.Button?>
<?import javafx.scene.control.Label?>
<GridPane fx:controller="sample.Controller"
          xmlns:fx="http://javafx.com/fxml" alignment="center" hgap="10" vgap="10">

    <Label fx:id="label" GridPane.columnIndex="0" GridPane.rowIndex="0"/>
    <Button fx:id="button" text="OK" onAction="#click" 
            GridPane.columnIndex="0" GridPane.rowIndex="1" GridPane.halignment="CENTER"/>

</GridPane>
```

ちなみに、以下のような感じでちょっとだけ入力すると補完候補が出てきます。
この辺はIntelliJ様々です。他のIDEもできるのかもしれないですけど。

![](/images/post_image_21.png)

次はControllerに動作を書いていきます。

``` java
package sample;

import javafx.fxml.FXML;
import javafx.scene.control.Button;
import javafx.scene.control.Label;

public class Controller
{
    @FXML
    private Label label;

    @FXML
    private Button button;

    public void click()
    {
        label.setText("ボタンをクリックしたよ！");
    }
}
```

コードの左側に表示されているマークをクリックすると、
@FXMLでマッピングしたコンポーネントにジャンプできます。
この辺もちょっとしたことですけど嬉しい機能です。

![](/images/post_image_22.png)

## 起動する
では、実際に起動してみます。
IntelliJでは右上に実行のアイコンが表示されているので、これをクリックします。
mainメソッドを持つクラスが自動的に認識されているので、特に設定は必要ないようです。

![](/images/post_image_23.png)

表示されたウィンドウの真ん中に表示されたボタンをクリックすると、
文字が表示されます。

![](/images/post_image_24.png)

## まとめ
本当はScene Builderとか使えば、レイアウトが楽だったりするのですが、
IntelliJだけでも補完機能が優れているので、何となくレイアウトできてしまったりします。
実は今回試してみるまで、`GridPane`のレイアウトで使う`GridPane.columnIndex`という属性は知らなかった...。
補完で出てきてくれたので、何となくこういう意味なんだろうなというノリで作れてしまいました。

というわけで、他のIDEの事情はちょっと分からないのですが、
IntelliJを使うことで、以外と簡単にJavaFXのアプリが作れることが分かって頂けると嬉しいなぁと思ったりしています。

最後に、IntelliJのAdvent CalendarじゃなくてJavaFXのAdvent Calendarです、念のため :P

Enjoy!

