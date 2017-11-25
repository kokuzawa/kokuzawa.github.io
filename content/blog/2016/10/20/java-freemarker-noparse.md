---
title: "FreeMarkerでinterpolation部分をそのまま出力"
date: 2016-10-20T13:41:21+09:00
tags: [Java, FreeMarker]
---
[FreeMarker-2.3.23](http://freemarker.org/)でinterpolation部分をそのまま出力したい。 

ただし、テンプレート文字列部分はユーザが自由に入力ができて、さらに、それがFreeMarkerのテンプレートだとは認識していない場合を想定。
つまり、ユーザが`${hello}`と入力したら、出力結果は`${hello}`となって欲しい。
調べてみると、下記のようにinterpolation部分を`${r"..."}`で括ればそのまま出力されるみたい。

**TEMPLATE:**

    ${r"${hello}"}

**OUTPUT:**

    ${hello}

ということは、ユーザの入力した文字列からinterpolation部分を抽出して、`${r"..."}`で括るように置換してあげればよさそうだけど、
ユーザが`${hello`としか入力しない場合に置換できないし、interpolation部分だけでなく、
`<#if>`などの制御タグもそのまま出力しなければならないので、この方法はあまり現実的ではなさそう。
で、FreeMarkerのマニュアルを眺めてみると、`noparse`という項があってそれをみたら「あ、これだ！」となった。
下記のように書くとそのまま出力される。

**TEMPLATE:**

```
<#noparse>
  <#if greet>
    ${hello}
  </#if>
</#noparse>
```

**OUTPUT:**

```
<#if greet>
  ${hello}
</#if>
```

これならユーザが入力した部分を`<#noparse>`で括ってしまえばいいだけなので簡単。

## 参考サイト

- [How to output ${expression} in Freemarker without it being interpreted?](http://stackoverflow.com/questions/5207613/how-to-output-expression-in-freemarker-without-it-being-interpreted)
- [FreeMarker.org#noparse](http://freemarker.org/docs/ref_directive_noparse.html)
