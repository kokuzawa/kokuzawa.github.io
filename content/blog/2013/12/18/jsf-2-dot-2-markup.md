---
title: "JSF 2.2 でさらに便利になったMarkupを使ってみよう"
date: 2013-12-17T00:12:00+09:00
tags: [Java, JSF] 
---
この記事は、[Java EE Advent Calendar 2013](http://www.adventar.org/calendars/152)の18日目の記事です。  
昨日は [@yumix_h](https://twitter.com/yumix_h) さんの[私がJava EE開発の現場から学んだこと](http://yumix.hatenablog.jp/entry/2013/12/17/235838)でした。  
明日は誕生日の [@aoetk](https://twitter.com/aoetk) さんです。

みなさんJSF使ってますか？  
JSFってちょっと取っ付きにくいところがありますよね。でもそんなJSFもバージョンが2.2になって、
ちょっと良い感じになってきたので使ってみませんか、
ということでJSFのマークアップについて書いてみたいと思います。 

## デザイナーとプログラマの作業の分担
JSFはデザイナーとプログラマで作業が分担できるということを良く聞きます。
デザイナーを雇うほど大きなプロジェクトでJSFを使ったことがないし、
分担しているということもあまり見聞きしませんが、実際この分担というのはどういうことなんでしょう？

デザイナーはHTMLは分かりますがJSFはわかりません。  
プログラマはHTMLもJSFも分かりますが、デザインセンスはそれを専属でやっているデザイナーのほうに
一日の長があると思うので、素直にデザインは任せた方が良いでしょう。

このようにHTMLしか分からない人が、HTMLに注力できるようにするには、
そのファイルがHTMLとして認識、つまりAPサーバーを経由しなくても、ブラウザでレイアウトが表示できる必要があります。
昔のJSPのように、JSPのタグやスクリプトレットで動的にレイアウトをするようにしていると、
ブラウザでレイアウトが表示できないのでダメということですね。

JSFはFaceletsというテンプレートエンジンを取り込んでいるので、
特別なファイルではなく、XHTMLとして画面を作ることができます。
そのため、JSFのコンポーネントを埋め込まなければ、このままブラウザで表示することができます。

ですが、何も埋め込まないのでは静的なHPと何も変わらないので、
JSFのコンポーネントを埋め込んでいく必要があり、下記のようなコンポーネントを埋め込んでしまうと、
デザイナーが理解できない状態、ブラウザでレイアウトを確認できない状態になってしまいます。

``` html
<h:commandButton value="OK" action=" #{managedBean.doAction}"/>
```

ではどうしたら良いのでしょう？

## JSF 2.1
JSF 2.1では、例えば下記のようにHTMLのタグにJSFのコンポーネント名を設定することによって、
JSFのコンポーネントとして認識させることができます。もちろんJSF 2.2でもできます。

``` html
<input type="submit" value="OK" jsfc="h:commandButton" action="#{managedBean.doAction}"/>
```

これをブラウザで表示してみます。

![](/images/post_image_25.png)

h:commandButtonの方は見事に表示されませんが、jsfc属性で書いた方は普通のボタンとして表示されています。

## JSF 2.2
jsfc属性でJSFコンポーネントをHTMLとして認識させるのには十分でしたが、
それぞれのJSFコンポーネント名をいちいち記載するのは面倒でした。コンポーネント名を正確に覚えていなければならないし...。
JSF 2.2ではそこがさらに改善され、下記のようにコンポーネント名を書かなくても、JSFコンポーネントとして認識してくれるようになりました。
namespaceは`xmlns:jsf="http://xmlns.jcp.org/jsf"`です。

``` html
<input type="submit" value="OK" jsf:action="#{managedBean.doAction}"/>
```

詳しくは[Java EE 7 Tutorial](http://docs.oracle.com/javaee/7/tutorial/doc/)を参考にしてください。

## ちょっと発展
ここまでで、JSFコンポーネントのマークアップが簡単だということが分かりました。
ただ、ビジネスアプリでは必ず登場するテーブルのJSFコンポーネントはちょっと考え方を変えないと、
HTMLとして認識させるのは難しそうです。

通常、JSFのテーブルコンポーネントは下記の形になります。

``` html
<h:dataTable var="item" value="#{managedBean.items}">
    <h:column>
        <f:facet name="header">ヘッダ１</f:facet>
        <h:outputText value="#{item.value1}"/>
    </h:column>
    <h:column>
        <f:facet name="header">ヘッダ２</f:facet>
        <h:outputText value="#{item.value2}"/>
    </h:column>
</h:dataTable>
```

これ、前から思ってたんですけど、HTMLのテーブルタグに慣れていると、
ちょっと直感的ではないのですぐに組み方を忘れてしまいませんか？
僕はいつも忘れてしまいます。
それにHTMLとして表示もできません。
これをHTMLとして表示できるように、なるべくテーブルタグに近づけていきたいと思います。

まずは通常のテーブルタグを書きます。ここまではデザイナーの分担ですね。

``` html
<table>
    <thead>
        <tr>
            <th>ヘッダ１</th>
            <th>ヘッダ２</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>値１</td>
            <td>値２</td>
        </tr>
    </tbody>
</table>
```

ここにJSFのコンポーネントを組み込んでいきます。

``` html
<table>
    <thead>
        <tr>
            <th>ヘッダ１</th>
            <th>ヘッダ２</th>
        </tr>
    </thead>
    <tbody>
        <tr jsfc="ui:repeat" var="item" value="#{managedBean.items}">
            <td>#{item.value1}</td>
            <td>#{item.value2}</td>
        </tr>
    </tbody>
</table>
```

tbodyのtrタグの部分にjsfc属性で`ui:repeat`を組み込みます。
JSF 2.2のjsfプレフックスは使えません。
こうすることでサーバー経由で表示した`h:dataTable`と同じになるはずです。
そうそう、ちゃんとHTMLとして表示されなければ意味がないので確認してみます。

![](/images/post_image_26.png)

分かりにくいのでボーダーを表示していますが、ちゃんとレイアウト通り表示されました。

## まとめ
こんな具合にJSFコンポーネントの情報を、HTMLのタグの属性として組み込めるようになっているので、
今までよりもずっとデザイナーとプログラマの作業の分担がやり易くなっていると思います。
まあ、デザイナーがいなくてもHTMLとして認識できるようにしておけば、
いちいちサーバーを起動しなくてもレイアウトが確認できるので、
JSFを使う場合は固有のタグではなく、HTMLの属性としてJSFを利用するようにした方が良いと思います。

## 補足
IntelliJ IDEA 13 でもjsfプレフィックスでの補完は完全には効かなかったので、
知らないとちょっと面倒かも :P

Enjoy!





