---
title: "Googleのテキスト読み上げAPIを組み込む（その２）"
date: 2015-05-07T15:34:12+09:00
tags: [Java, JAX-RS] 
---

先日書いたコードがiOSのSafariで上手く動きませんでした。

[Googleのテキスト読み上げAPIを組み込む](http://kokuzawa.github.io/blog/2015/05/03/translate-google/)

子供向けなのでiPhoneまたはiPadで音声が再生されないと困ります。
いろいろ調べたけれど、JavaScriptから呼び出す方法がわからないので
勝手知ったるJavaの世界に取り込んでiOSのSafariでも音声が再生されるようにしました。

<!-- MORE -->

下記がそのコードです。

``` java
@Path("/tts")
public class TTSResource
{
    @GET
    @Produces("audio/mpeg")
    public Response textToSpeech(@QueryParam("text") String text) throws IOException
    {
        final URL url = new URL("http://translate.google.com/translate_tts?tl=en&q=" + text);
        final URLConnection connection = url.openConnection();
        connection.setRequestProperty("User-Agent", "Mozilla");
        return Response.ok(connection.getInputStream()).build();
    }
}
```

JAX-RSでGoogle Translateの結果をそのままレスポンスとして返すようにしています。
User-Agentを指定していないと上手く動きません。
ここで気がついたのですがUser-Agentで振る舞いが変わるようなので、
もしかしたらiOSからのアクセスの場合にもUser-Agentを偽装できれば音声が再生されるのかもしれないです。

先日の記事にも書きましたがクライアント側（javaScript）は下記のようになります。

``` html
<h:outputScript>
function play() {
    var voice = new Audio();
    voice.src = "${request.contextPath}/rest/tts?text=one";
    voice.play();
}
</h:outputScript>

<a href="javascript:play()">音声</a>
```


