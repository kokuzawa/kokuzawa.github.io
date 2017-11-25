---
title: "Server Sent Events"
date: 2015-12-20T10:32:29+09:00
tags: [Java, JAX-RS, JavaEE] 
---
これは [JavaEE Advent Calendar 2015](http://qiita.com/advent-calendar/2015/javaee) の20日目の記事です。  
昨日は[@yumix_h](https://twitter.com/yumix_h)さんの「[「帰ってきたGlassFish Users Group Japan勉強会」の未発表資料](http://yumix.hatenablog.jp/entry/2015/12/19/205954)」でした。  
明日は[@emaggame](https://twitter.com/emaggame)さんです。

<!-- MORE -->

## Server Sent Eventsとは

Server Sent Events (SSE) はサーバから送られたイベントという意味の通り、push型のデータ通信を行うことができます。
これはHTML5で追加された新機能です。
同じくpush型のデータ通信を行う方法としてWebsocketがありますが、WebsocketがHTTPとは別のプロトコルで通信をするのに対し、
SSEではHTTPプロトコルを利用します。そのため、既存のHTTPを利用した通信との互換性が高いというメリットがある反面、
Websocketのような双方向の通信を行うことはできません。
HTTPプロトコルでpush通信を実現するため、SSEではサーバからのレスポンスを受けても接続を終了せずに継続させます。
こうすることで、サーバ側からのデータを継続して受信することを実現します。
このようにSSEはHTTPプロトコルで接続を行うのですが、クライアントがSSEだと認識できるデータを送ってもらう必要があります。
そこで、サーバはMIMEタイプに`text/event-stream`を設定する必要があります。

JavaEE8にSSEのサポートが入るようですが、一足先にJAX-RSのRIであるJerseyでこの機能を試すことができます。

## Server Sent Eventsを試す

今回実行した環境は下記の通りです。

- OS: Mac OSX 10.11.1
- Java: Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
- APサーバ: GlasshFish-4.1.1
- ブラウザ: Safari-9.0.1

実際のコードはGithubにあるので、
コードを見れば分かる方は以降の実装の説明を読むより、
そちらを見ていただいた方が早いかと思います。

[sandbox/sse-example](https://github.com/kokuzawa/sandbox/tree/master/sse-example)

## 実装の説明

Mavenを利用しているので、最初に下記のDependencyを追加します。
2015/12/10時点のMaven Centralの最新版は2.22.1のようです。

```xml
<dependency>
  <groupId>org.glassfish.jersey.media</groupId>
  <artifactId>jersey-media-sse</artifactId>
  <version>2.22.1</version>
</dependency>
```

サーバ側のリソースはMIMEタイプに`text/event-stream`を設定する他に、
`org.glassfish.jersey.media.sse.EventOutput`を返却する必要があります。

```java
@GET
@Produces(SseFeature.SERVER_SENT_EVENTS)
public EventOutput getServerSentEvents()
{
    ...
}
```

`EventOutput`を返却するだけだと、クライアントとの接続が確立しているだけの状態なので、
実際にクライアントに送信するデータを書き込む必要があります。
書き込みは`EventOutput#write(OutboundEvent)`で行います。
単純には下記のような実装になります。

```java
final EventOutput eventOutput = new EventOutput();
final OutboundEvent.Builder builder = new OutboundEvent.Builder();
builder.name("message-to-client");
builder.data(String.class, "Hello world !");
eventOutput.write(builder.build());
```

`builder.name(...)`で指定している文字列はクライアント側でイベントのマッピングをするために利用します。

今回クライアントはJavascriptにします。
JavascriptでSSEを利用するには`EventSource`クラスを利用します。
`EventSource`を利用した実装は下記のようになります。

```javascript
var eventList = document.getElementById("eventList");
var eventSource = new EventSource("http://localhost:8080/sse-example/api/sse/events");
eventSource.addEventListener("message-to-client", function (e) {
    var newElement = document.createElement("li");
    newElement.innerHTML = "message: " + e.data;
    eventList.appendChild(newElement);
});
```

EventSourceコンストラクタの引数でAPIエンドポイントを指定します。
addEventListenerでサーバからのイベントをハンドリングします。
この時、リスナーに設定するイベント名として、サーバ側コードで指定したイベント名を指定します。
この例では"message-to-client"です。

さて、実際のコードの説明です。  
ユースケースとして複数のユーザがそれぞれブラウザの画面を表示している状態で、
データが登録されると、開いている画面に登録された旨を伝えるメッセージを表示することを考えます。
まず必要なのは接続を確立するために`EventOutput`を返却するサービスです。
`EventOutput`はクライアントごとにインスタンスが必要なので、
接続が確立した`EventOutput`を格納するためのリストも合わせて定義します。
これらを踏まえて下記のコードを作ります。

```java
private List<EventOutput> eventOutputs = new ArrayList<>();

@GET
@Path("events")
@Produces(SseFeature.SERVER_SENT_EVENTS)
public EventOutput getServerSentEvents()
{
    final EventOutput eventOutput = new EventOutput();
    eventOutputs.add(eventOutput);
    return eventOutput;
}
```

次に登録をするためのサービスを作ります。が、実際に何かを登録するのは実装が面倒なので、
サービスが呼ばれたら各クライアントにメッセージをpushするだけにします。
こんな感じです。

```java
@PUT
@Path("put")
public void putData() throws IOException
{
    for (EventOutput eventOutput : eventOutputs) {
        final OutboundEvent.Builder builder = new OutboundEvent.Builder();
        builder.name("message-to-client");
        builder.data(String.class, "登録された！");
        eventOutput.write(builder.build());
    }
}
```

これでサービス側は実装完了です。
Javascriptクライアントを実装する前に正しく動くかcurlコマンドで確認してみます。
接続確立のサービスを下記のように呼び出します。

    curl http://localhost:8080/sse-example/api/sse/events

プロンプトが待ち状態になりました。接続されたままになったのでうまくいったようです！
別のプロンプトから次のコマンドを実行して最初のプロンプトに通知されるか確認します。

    curl -X PUT http://localhost:8080/sse-example/api/sse/put

最初のプロンプトの方に以下のメッセージが表示されました。こちらもうまくいったようです。

    event: message-to-client
    data: 登録された！

サービス側が正常に動作することが確認できたので、
次にJavascriptクライアントを作ります。
HTMLを含めた全コードは下記のようになりました。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SSE Example</title>
    <script>
        function startup() {
            var eventList = document.getElementById("eventList");
            var eventSource = new EventSource("http://localhost:8080/sse-example/api/sse/events");
            eventSource.addEventListener("message-to-client", function (e) {
                var newElement = document.createElement("li");
                newElement.innerHTML = "message: " + e.data;
                eventList.appendChild(newElement);
            });
        }
    </script>
</head>
<body onload="startup()">
    <h1>イベント表示:</h1>
    <ul id="eventList"></ul>
</body>
</html>
```

## まとめ

このようにSSEの実装は比較的簡単に行うことがでます。
ただ最初にも書いたようにSSEは一方向通信なので、push通信だけでなく双方向通信を行いたい場合は
Websocketを利用することになります。
利用シーンとしてはWebsocketの方が多くなりそうですが、
既存のアプリにpush通知機能を実装するという観点からであれば、
HTTPプロトコルで動作するSSEを利用した方が良いケースがあるかもしれないですね。

## 参考にしたサイト

- [Chapter 15. Server-Sent Events (SSE) Support](https://jersey.java.net/documentation/latest/sse.html)

