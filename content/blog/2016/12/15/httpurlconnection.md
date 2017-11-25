---
title: "HttpURLConnectionで嵌った話"
date: 2016-12-15T00:03:24+09:00
tags: [Java]
---
この記事は[Java Advent Calendar 2016](http://qiita.com/advent-calendar/2016/java)の15日目です。  
昨日は[enk](http://qiita.com/enk)さんの「[JGiven で 100% Pure Java BDD（導入編）](http://qiita.com/enk/items/240e56a00e104d7088d8)」でした。

<!-- more -->

HttpURLConnectionには`getInputStream`と`getErrorStream`というサーバからのレスポンスを受け取るためのメソッドが用意されています。
この二つのメソッドのうち、`getErrorStream`のJavadocを見ると下記のように記載されています。

> 接続が失敗したが、それにもかかわらずサーバーから有用なデータを送信されてきた場合に、エラー・ストリームを返します。典型的な例としては、HTTPサーバーが404で応答し、それによって接続内でFileNotFoundExceptionがスローされたが、そのサーバーから対処策を含むHTMLヘルプ・ページが送信されてきた、といった場合です。

これを読むと少なくともステータスコードが404の場合にはエラーストリームが取得できそうな気がするのですが、
実際のところインプットストリームで返却するのかエラーストリームで返却するのか明確に仕様が決まっているわけではないらしく、
接続先のサーバの実装に依存し、取得できたりできなかったりします。

インプットストリームで返却されたのかエラーストリームで返却されたのか、
事前に判定するための方法が用意されているわけでもないため、
実際には下記のようなコードでストリームを取得する必要がありそうです。
インプットストリームが取れない場合は`IOException`が発生、
エラーストリームが取れない場合はnullが返却されます。

エラーストリームが取れない場合にインプットストリームを取得:

```java
InputStream stream = connection.getErrorStream();
if (null = stream) {
    stream = connection.getInputStream();
}
```

インプットストリームが取れない場合にエラーストリームを取得:

```java
InputStream stream;
try {
    stream = connection.getInputStream();
}
catch (IOException e) {
    stream = connection.getErrorStream();
}
```

## 実際の問題

JAX-RSクライアントライブラリのresteasy-client 3.0.10が持つクラス、
`org.jboss.resteasy.client.jaxrs.engines.URLConnectionEngine`を利用した際、
サーバが4xxのステータスコードを返却するとNullPointerExceptionが発生します。
URLConnectionEngineの該当箇所のコードは下記のようになっています。

```java
@Override
protected InputStream getInputStream()
{
    if (stream == null)
    {
        try
        {
            stream = (status < 300) ? 
                    connection.getInputStream() : connection.getErrorStream();
        }
        catch (IOException e)
        {
            throw new RuntimeException(e);
        }
    }

    return stream;
}

@Override
protected void releaseConnection() throws IOException
{
    getInputStream().close();
    connection.disconnect();
}
```

ステータスコードが300未満の場合はインプットストリーム、300以上の場合はエラーストリームを取得し、
その取得したストリームをクローズしようとしたところでNullPointerExceptionが発生する状況です。
このクライアントコードを書いた人は、ステータスコードが300以上の場合はエラーと判断したのだと思います。
ところが実際はステータスコードが4xxが返却されてもエラーストリームはnullになっていました。

## まとめ

結局、インプットストリームを返却するのかエラーストリームを返却するのか、
仕様として明確に決まっていないために、サーバの実装とクライアントの実装が一致せずに問題が発生しているのだと思います。
少なくともステータスコードで判断することはできないので、
最初にあげたように泥臭いコードでストリームを取得しなければならないのでしょう。
HttpURLConnectionクラスに判定メソッドが追加されると良いとは思うのですが、
Java8の段階ではそのようなメソッドは見当たらないです。

ちなみにresteasy-clientはというと、3.0.15でこの問題は修正されています。

https://github.com/resteasy/Resteasy/blob/master/resteasy-client/src/main/java/org/jboss/resteasy/client/jaxrs/engines/URLConnectionEngine.java


