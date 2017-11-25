---
title: "remoteCommand in Composite"
date: 2016-12-14T01:28:24+09:00
tags: [Java, JSF]
---
JSFにはcompositeというカスタムコンポーネントを作るための仕組みがあります。
PrimeFaces-6.0にはManagedBeanのメソッドを呼び出すためのremoteCommandというコンポーネントがあります。
この二つを使ってカスタムコンポーネントを作ったところ、ManagedBeanのメソッドが呼ばれないという問題が発生しました。

<!-- more -->

まず、JSFのcomposite機能を使って下記のような二つのコンポーネントを作りました。

sample1.xhtml:

``` xml
<composite:implementation>
  <p:tree>
    <p:ajax event="select" oncomplate="afterSelected()"/>
  </p:tree>
  <p:remoteCommand name="afterSelected" actionListener="#{bean.method1}"/>
</composite:implementation>
```

sample2.xhtml:

``` xml
<composite:implementation>
  <p:tree>
    <p:ajax event="select" oncomplate="afterSelected()"/>
  </p:tree>
  <p:remoteCommand name="afterSelected" actionListener="#{bean.method2}"/>
</composite:implementation>
```

そしてこれらを一つのXHTMLに組み込みます。

main.xhtml:

``` xml
<my:sample1/>
<my:sample2/>
```

sample1側のツリーノードを選択した際に`bean.method1`が呼ばれることを想定していたのですが、
呼ばれることなく画面がリフレッシュされました。それぞれのカスタムコンポーネント内の
remoteCommandのnameの値が重複していると、エラーが発生することなくメソッドが呼ばれないという現象が発生します。

当たり前と言えば当たり前なのですが、
似たようなコンポーネントを作るとやらかしてしまいそうなので注意しないと。

