---
title: "Googleのテキスト読み上げAPIを組み込む"
date: 2015-05-03T22:12:37+09:00
tags: [HTML, Java, JSF] 
---
子供用の英単語帳Webアプリを作っています。
単語の読み上げ機能があると便利だなと思い、そんなAPIはないかと調べてみると、
Googleが提供しているテキスト読み上げAPIというのを見つけました。

使い方はとても簡単で、下記のようなURLを呼び出せば良いようです。
この例ではoneという単語を読み上げてくれます。

    http://translate.google.com/translate_tts?tl=en&q=one

<!-- MORE -->

これをHTMLに組み込みます。
HTMLで音声を再生するためにはaudioタグを利用します。
下記のように組み込むことで音声を再生するためのコントロールを表示することができます。

    <video src="http://translate.google.com/translate_tts?tl=en&q=one" controls />

こんな感じになります。

<video src="http://translate.google.com/translate_tts?tl=en&q=one" controls />

実際のアプリはJSF-2.2でFaceletを利用しています。
このタグを組み込んだ場合、最初の一回目は正しく再生されますが
2回目以降が再生されません。
なぜダメなのか結局分からなかったのですが、
UI的にはaudioコントロールを表示したくなかったので、
タグを埋め込むのではなくJavaScriptで再生するようにしました。

``` html
<script>
function play() {
  var audio = document.createElement("audio");
  audio.src = "http://translate.google.com/translate_tts?tl=en&q=one";
  audio.play();
}
</script>

<a href="#" onClick="play()">再生</a>
```

この場合audioコントロールは表示されないのですが、
再生リンクをクリックするたびに正しく再生されます。

