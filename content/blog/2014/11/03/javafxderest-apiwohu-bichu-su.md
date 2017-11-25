---
title: "FlexとJavaFXでREST-APIを呼び出す"
date: 2014-11-03T17:12:02+09:00
tags: [Java, JavaFX, JAX-RS] 
---
FlexとJavaFXからREST-APIを呼び出してみました。

<!-- MORE -->

## 環境

- OS: Mac OSX Yosemite 10.10
- Java: Java(TM) SE Runtime Environment (build 1.8.0-b132)
- Flex SDK 4.6
- メモリ: 4GB
- WildFly 8.0.0.Final

## サーバの用意

今回の本題ではないので、ここでは簡単な文字列を返すだけのAPIを作成します。
引数で受け取った文字を加工して「Hello XXX!」という文字を返します。
RESTの実装にはJersey-2.8を利用します。

``` java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class HelloService
{
    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello(@QueryParam("string") String string)
    {
        return String.format("Hello %s!", string);
    }
}
```

## FlexからのREST-API呼び出し
JavaFXとの比較のためにFlexからのREST-API呼び出しを提示します。

画面構成を管理するMXMLです。

**Main.mxml:**

``` xml
<?xml version="1.0"?>
<s:WindowedApplication
        xmlns:fx="http://ns.adobe.com/mxml/2009"
        xmlns:s="library://ns.adobe.com/flex/spark"
        xmlns:local="*" 
        title="Hello World" 
        width="230" 
        height="80">
    <fx:Declarations>
        <local:Controller id="controller"/>
    </fx:Declarations>
    <s:VGroup paddingBottom="10" 
              paddingLeft="10" 
              paddingRight="10" 
              paddingTop="10" 
              gap="10" 
              minHeight="0">
        <s:HGroup>
            <s:TextInput id="stringField"/>
            <s:Button label="Button" click="{controller.buttonAction(event)}"/>
        </s:HGroup>
        <s:Label id="stringLabel"/>
    </s:VGroup>
</s:WindowedApplication>
```

画面をコントロールするコントローラクラスです。

**Controller.as:**

``` as
package {
    import flash.events.MouseEvent;

    import mx.core.IMXMLObject;
    import mx.rpc.events.ResultEvent;
    import mx.rpc.http.mxml.HTTPService;

    public class Controller implements IMXMLObject
    {
        private var _document:Main;

        public function initialized(document:Object, id:String):void
        {
            _document = document as Main;
        }

        public function buttonAction(event:MouseEvent):void
        {
            var service:HTTPService = new HTTPService("http://localhost:8080");
            service.url = "/jaxrs/rest/hello";
            service.addEventListener(ResultEvent.RESULT, function (e:ResultEvent):void
            {
                _document.stringLabel.text = e.result as String;
            });
            service.send({string:_document.stringField.text});
        }
    }
}
```

起動すると下記の画面が表示されます。

![](/images/post_image_37.png)

## JavaFXからREST-APIを呼び出す
JavaFXからの呼び出し例を提示します。

画面構成を管理するFXMLです。
画面レイアウトにはSceneBuilder-2.0を利用しました。

**sample.fxml:**

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.geometry.Insets?>
<?import javafx.scene.control.Button?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.control.TextField?>
<?import javafx.scene.layout.*?>
<GridPane alignment="center" 
          hgap="10" 
          prefHeight="46.0" 
          prefWidth="324.0" 
          vgap="10" 
          xmlns="http://javafx.com/javafx/8" 
          xmlns:fx="http://javafx.com/fxml/1" 
          fx:controller="sample.Controller">
   <columnConstraints>
      <ColumnConstraints />
      <ColumnConstraints minWidth="10.0" prefWidth="60.0" />
   </columnConstraints>
   <rowConstraints>
      <RowConstraints />
      <RowConstraints minHeight="10.0" prefHeight="30.0" />
   </rowConstraints>
   <children>
      <TextField fx:id="stringField" prefHeight="26.0" prefWidth="205.0" />
      <Button mnemonicParsing="false" text="Button" GridPane.columnIndex="1" onAction="#buttonAction"/>
      <Label fx:id="stringLabel" GridPane.columnSpan="2" GridPane.rowIndex="1" />
   </children>
   <padding>
      <Insets bottom="10.0" left="10.0" right="10.0" top="10.0" />
   </padding>
</GridPane>
```

JavaによるREST-API呼び出しは、JAX-RSクライアントを利用するため、
下記ライブラリを追加します。（Mavenの設定）

``` xml
<dependencies>
    <dependency>
        <groupId>org.glassfish.jersey.core</groupId>
        <artifactId>jersey-client</artifactId>
        <version>2.8</version>
    </dependency>
</dependencies>
```

画面をコントロールするコントローラクラスです。
ボタンがクリックされた場合にREST-APIを呼び出して結果をラベルに設定します。

**Controller.java:**

``` java
import javafx.fxml.FXML;
import javafx.scene.control.Label;
import javafx.scene.control.TextField;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.MediaType;

public class Controller
{
    @FXML
    private TextField stringField;

    @FXML
    private Label stringLabel;

    public void buttonAction()
    {
        final Client client = ClientBuilder.newClient();
        try {
            final String result = client.target("http://localhost:8080/jaxrs/rest")
                    .path("hello")
                    .queryParam("string", stringField.getText())
                    .request(MediaType.TEXT_PLAIN_TYPE)
                    .get(String.class);
            stringLabel.setText(result);
        }
        finally {
            client.close();
        }
    }
}
```

起動すると下記の画面が表示されます。

![](/images/post_image_38.png)

## JavaFXからの呼び出しを非同期にする
FlexもJavaFXもほぼ同じようなコードでREST-APIを呼び出すことができるのですが、
Flexの方は非同期呼び出しであり、JavaFXの方は同期呼び出しという違いがあります。
そこでJavaFXの方でも非同期呼び出しをさせてみたいと思います。

JAX-RSクライアントには非同期呼び出しの仕組みがあるので、
それを利用するようにREST-APIの呼び出し部分を下記のように書き換えました。

``` java
final Client client = ClientBuilder.newClient();
client.target("http://localhost:8080/jaxrs/rest")
      .path("hello")
      .queryParam("string", stringField.getText())
      .request(MediaType.TEXT_PLAIN_TYPE)
      .async()
      .get(new InvocationCallback<String>()
      {
          @Override
          public void completed(String result)
          {
              stringLabel.setText(result);
          }

          @Override
          public void failed(Throwable throwable)
          {
              throwable.printStackTrace();
          }
      });
```

呼び出しチェーンに`async()`メソッドを追加します。
結果は戻り値ではなく、`InvocationCallback<T>`インターフェースの`completed(T)`メソッドで受け取るようになります。
また、呼び出し後にclientをクローズしてしまうと非同期によるレスポンスを受け取る前に接続が切れてしまいます。
そのため、ここではクローズは行いません。

これで非同期になると思いきや、実行すると下記エラーが発生します。

``` java
javax.ws.rs.ProcessingException: java.lang.IllegalStateException: Not on FX application thread; currentThread = jersey-client-async-executor-0
	at org.glassfish.jersey.client.ClientRuntime.processFailure(ClientRuntime.java:173)
	at org.glassfish.jersey.client.ClientRuntime.access$400(ClientRuntime.java:69)
	at org.glassfish.jersey.client.ClientRuntime$1.run(ClientRuntime.java:155)
```

JavaFXアプリのスレッド以外でアクセスしようとしたのでエラーが発生しています。
というわけで`async()`メソッドは使えません。
そこで、`javafx.concurrent.Service`クラスを利用します。
このクラスを利用すると、JavaFXアプリにおいて別スレッドを利用できるようになります。

`javafx.concurrent.Service`クラスを利用したクラスが下記になります。

**HelloService.java:**

``` java
import javafx.beans.property.SimpleStringProperty;
import javafx.beans.property.StringProperty;
import javafx.concurrent.Service;
import javafx.concurrent.Task;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.MediaType;

public class HelloService extends Service<String>
{
    private StringProperty string = new SimpleStringProperty();

    public StringProperty stringProperty()
    {
        return string;
    }

    @Override
    protected Task<String> createTask()
    {
        return new Task<String>()
        {
            @Override
            protected String call() throws Exception
            {
                if (false == string.get().isEmpty()) {
                    final Client client = ClientBuilder.newClient();
                    try {
                        return client.target("http://localhost:8080/jaxrs/rest")
                                .path("hello")
                                .queryParam("string", string.get())
                                .request(MediaType.TEXT_PLAIN_TYPE)
                                .get(String.class);
                    }
                    finally {
                        client.close();
                    }
                }
                return null;
            }
        };
    }
}
```

作成したサービスクラスを利用するようにコントローラクラスを書き換えます。

**Controller.java:**

``` java
import javafx.fxml.FXML;
import javafx.fxml.Initializable;
import javafx.scene.control.Label;
import javafx.scene.control.TextField;

import java.net.URL;
import java.util.ResourceBundle;

public class Controller implements Initializable
{
    @FXML
    private TextField stringField;

    @FXML
    private Label stringLabel;

    private HelloService service = new HelloService();

    public void buttonAction()
    {
        service.restart();
    }

    @Override
    public void initialize(URL url, ResourceBundle resourceBundle)
    {
        service.stringProperty().bind(stringField.textProperty());
        service.setOnSucceeded(e -> stringLabel.setText((String)e.getSource().getValue()));
        service.start();
    }
}
```

`Initializable`インターフェースの`initialize(URL, ResourceBundle)`メソッド内で
入力フィールドをサービスクラスへバインドし、サービスの処理完了時に呼ばれる`setOnSucceeded`メソッドで
ラベルに対してレスポンスを書き出すようにします。`start()`メソッドでサービスを開始します。

また、書き換え前のコードではボタンクリックのハンドラ内でREEST-APIを呼び出していましたが、
新しいコードではサービスクラスの`restart()`メソッドを呼び出し、
サービスの起動状態をキャンセルして再起動させます。こうすることにより、サービス内のタスクが再度生成されるので、
入力された値がサーバーに送信されるようになります。
入力値はバインドを利用しているので、サービスへの再設定は必要ありません。

## まとめ

FlexとJavaFXでの簡単な呼び出しにおいてはほとんど違いがないことがわかるかと思います。
JavaFXで非同期呼び出しをしようとした場合にちょっとだけ面倒になりますが、
非同期にしたい部分だけ今回のようにサービスにするだけなので、
JavaFXを利用する上ではそれほど問題にならないかな、と思っています。

Flexでは同期呼び出しにするという選択肢がないので、
同期と非同期を切り替えられるJavaFXの方がメリットがありそうです。

## 参考

- [Concurrency in JavaFX](http://docs.oracle.com/javafx/2/threads/jfxpub-threads.htm)

