---
title: "Dialog of PrimeFaces"
date: 2016-12-25T15:10:00+09:00
tags: [Java, JSF, PrimeFaces]
---
この記事は[Java EE Advent Calendar 2016](http://qiita.com/advent-calendar/2016/javaee)の25日目です。  
昨日は[@kikutaro](http://qiita.com/kikutaro)さんの「[実はJava EEに含まれるJavaMailについて](http://qiita.com/kikutaro/items/a524038ca7306a8bfee3)」でした。

<!-- more -->

現在業務でJSFを使っています。
導入当初はRIであるMojarraのみを利用しようと考えていたのですが、
業務アプリで多い、ツリーやグリッドで数多くのアクションを実装しなければならず、
一つ一つをJavaScriptで実装していくには時間が足りないという判断のもとに、
それらを簡易に実現できるPrimeFacesを利用することにしました。
採用を決定した段階での最新バージョンはPrimeFaces-6.0です。
PrimeFacesは充実したコンポーネント群を持っているので、
必要なコンポーネントはほぼ見つけることができるかと思います。

さて、今回はその中でダイアログコンポーネントについて説明します。
PrimeFacesのDemoを見るとわかるのですが、
このダイアログコンポーネントを表示するための方法が二通り用意されています。

一つ目は静的にダイアログを表示する方法です。

**XHTML:**

```xml
<p:dialog widgetVar="sampleDialog">
  ...
</p:dialog>

<p:commandButton value="Show" oncomplete="PF('sampleDialog').show()"/>
```

二つ目の方法は動的にダイアログを表示する方法です。

**ManagedBean:**

```java
public void onShowDialog()
{
    RequestContext.getCurrentInstance().openDialog("dialog.xhtml");
}
```

**XHTML:**

```xml
<p:commandButton value="Show" actionListener="#{bean.onShowDialog}"/>
```

二つ目の方法は指定したXHTMLをiframe内に表示して、それをダイアログとして表示してくれます。
一つ目の方法と異なり、
ダイアログ内のコンテンツを別XHTMLに分けることができるのでコードの見通しが良くなるかと思います。
また、表示時にダイアログのオプションを指定することができますが、
何も指定しないと、モーダレス、リサイズ可能、コンテンツが640pxで固定されたダイアログが表示されます。
ダイアログをリサイズしてもコンテンツが640pxで固定されているので、追従して広がることがありません。
もし、リサイズに合わせてコンテンツも追従するようにしたければ、
表示時に下記のようなオプションを付与します。

```java
public void onShowDialog()
{
    final Map<String, Object> options = new HashMap<>();
    options.put("width", 640);
    options.put("contentWidth", "100%");
    RequestContext.getCurrentInstance().openDialog("dialog.xhtml", options, null);
}
```

このようにすることで、ダイアログの初期表示の幅は640px、コンテンツの幅は100%となり、
コンテンツがリサイズに追従するようになります。

さらにパラメータを渡すことも可能です。

```java
public void onShowDialog()
{
    final Map<String, List<String>> params = new HashMap<>();
    params.put("id", Collections.singletonList("123"));
    RequestContext.getCurrentInstance().openDialog("dialog.xhtml", null, params);
}
```

パラメータはiframeのsrc属性に付与され、`/contextPath/dialog.xhtml?id=123`という値になります。

## ちょっとしたコツ

ここまではドキュメントを読めば大体の内容は記載されているのですが、
以下は実体験に伴う内容です。

ダイアログは簡単に表示できますが、
コンポーネントの構成次第で最初の一回は表示され、2回目以降は表示されないという問題が発生します。
具体的には下記のような構成とした場合です。

```xml
<p:panel rendered=”#{bean.showPanel}”>
    <p:commandButton value="Show" actionListener="#{bean.onShowDialog}"/>
</p:panel>
```

親コンポーネントにrendered属性が付与されている場合、
2回目以降のダイアログの表示が行われません。
バグの可能性もありますが、現状での回避策は下記のように、
rendered属性が付与された親コンポーネントの外に隠しボタンを用意し、
実際のボタンがクリックされた際に隠しボタンをクリックするようにします。

```xml
<p:commandButton id="hiddenButton" actionListener=”#{bean.onShowDialog}” 
                 style="visibility: hidden;"/>
<p:panel rendered="#{bean.showPanel}">
    <p:commandButton value="Show" oncomplate="$('#hiddenButton').click()"/>
</p:panel>
```

コンポーネントの組み合わせによっては落とし穴もあるよ、ということでした。

## 参考

- [PrimeFaces ShowCace](http://www.primefaces.org/showcase/ui/df/data.xhtml)

