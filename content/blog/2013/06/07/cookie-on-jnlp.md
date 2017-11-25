---
title: "cookie on java web start (JWS)"
date: 2013-06-07T01:41:00+09:00
tags: [Java]
---
今更感がありますが、JWSでのcookieの扱い、というかjsessionidの扱いについて。

JWSからサーバへの最初のアクセスでjsessionidの指定が無いと、サーバでjsessionidが発行されます。
すると、JWSからのそれ以降のアクセスには、その生成されたjsessionidがcookieとしてサーバに送信されます。
これは良く出来た仕組みで、プログラマが意識しなくてもセッションを継続することが可能になります。
ただ、自動でjsessionidを送信してしまうため、意図したセッションを継続させることが出来ないことがあります。

<!-- MORE -->

## 任意のjsessionidを送信したい
前提として以下のことが成り立っているとします。

* 認証はプラウザで行う
* 認証状態をセッションで維持している

JWSでもその認証状態を維持したい、つまり認証されていなければJWSを操作させたくない場合、
簡単な方法としてJWS起動時にsessionidを渡せば、JWSからのリクエストにjsessionidを付与することが可能になり、
認証状態のあるセッションを継続することが出来るようになります。

例えば、以下のようにJWSからHttpURLConnectionを利用してリクエストを送信するケースですが、
`HttpURLConection#setRequestProperty(String, String)`を使うことによってcookieにjsessionidを含めることが可能です。
もちろん、URLの最後に`;jsessionid=XXXXX`という形でjsessionidを付けても問題ありません。
ちなみに、URLの最後に付けるのと、`setRequestProperty`での設定と両方の指定がある場合は、
`setRequestProperty`の方が優先されます。

``` java
final HttpURLConnection connection = 
    (HttpURLConnection) new URL("http://localhost/mycontext").openConnection();
connection.setRequestProperty("Cookie", "JSESSIONID=" + sessionid);
```

このリクエストがJWSからの最初のリクエストでは無く、かつ最初のリクエストでjsessionidを明示的に送っていない場合、
Cookieには2つのjsessionidが設定された状態でリクエストされます。
サーバではこの2つのjsessionidのうち、最初に設定されている方を有効と見なして処理が行われます。
よって、上記コードで明示的に設定したjsessionidが無視されることになります。

では、このケースで明示的に設定したjsessionidを有効にするにはどうしたら良いのか。

## 自動で設定されるcookieを無効にする
自動で設定されるcookieは`CookieHandler`で設定されるようです。
なので、この`CookieHandler`を機能させなくすれば、cookieが自動で設定されなくなるはず。
というわけで、下記のコードを追加します。

``` java
CookieHandler.setDefault(null);
```

これでcookieの自動設定が行われないはずなのですが、実はこのメソッド、セキュリティが
all-permissionsでないと、`AccessControlException`がスローされてしまいます。
JWSでall-permissionsにするには、jarファイルに署名しなければなりません。

## 最後に
jarファイルに署名できないような政治的な事情がある場合はこの方法が取れないので、
JWSからの最初のリクエストにちゃんとjsessionidが付与されるようにしましょう。

まあ、署名するための証明書もオレオレ証明書じゃない場合は、
それなりに費用もかかるので、署名できないことも多い訳ですが... :P

それにしても、JNLPのこととか、JWSなどの記事が軒並み古いので、
もうほとんど使われてないのかなぁと思ったりした快晴の木曜日でした。

enjoy!

