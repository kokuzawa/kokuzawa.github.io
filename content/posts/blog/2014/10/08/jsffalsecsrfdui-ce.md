---
title: "JSFのCSRF対策"
date: 2014-10-08T02:29:57+09:00
tags: [Java, JSF] 
---
だいぶ前にリリースされたJSF 2.2ではCSRF対策の機能が追加されました。
そこで、JSFをどのように利用している場合にCSRFの脆弱性が発生するのかと、
JSF 2.2で追加されたCSRF対策を実施すると、この問題をどのように防げるのかを確認してみます。

<!-- MORE -->

## 環境

- OS: Mac OSX 10.9.4
- Java: Java(TM) SE Runtime Environment (build 1.8.0-b132)
- メモリ: 4GB
- WildFly 8.0.0.Final

## アプリを作る

CSRFの脆弱性がある、といわれても具体例がないとなかなかイメージするのが難しいかもしれません。
そこで、ここでは実際に攻撃を受けるアプリを作り、脆弱性を露呈されてみたいと思います。
コードの構成は下記のようになります。

    JSFCSRFSample
        +- src
        |  +- main
        |  |   +- java
        |  |       +- org.katsumi.bean
        |  |           +- FormBean.java
        |  +- webapp
        |      +- index.xhtml
        |      +- result.xhtml
        |      +- warning.xhtml
        |      +- WEB-INF
        |          +- jboss-web.xml
        |          +- faces-config.xml
        |          +- web.xml
        +- pom.xml

index.xhtmlページでボタンをクリックすることで、result.xhtmlページに遷移します。
ユースケースとして、result.xhtmlでindex.xhtmlから受け取ったパラメータを登録して、その値を表示することを想定します。
サンプルコードでは、index.xhmlから送信したパラメータをFormBean.javaで受け取り、
その値を加工して結果を表示しています。
index.xhtml、result.xhtml、FormBean.javaは下記のように非常にシンプルな構成です。

**index.xhtml:**

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<f:view transient="true">
    <h:button value="Jump!" outcome="/result?text=ああああ"/>
</f:view>
</html>
```

**result.xhtml:**

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<f:view>
    <f:metadata>
        <f:viewParam name="text" value="#{formBean.text}"/>
    </f:metadata>
    <p>結果を表示:</p>
    <h:outputText value="#{formBean.text}"/>
</f:view>
</html>
```

**FormBean.java:**

``` java
package org.katsumi.bean;

import lombok.Setter;
import javax.enterprise.context.RequestScoped;
import javax.inject.Named;

@Named
@RequestScoped
public class FormBean
{
    @Setter
    private String text;

    public String getText()
    {
        return String.format("登録しました「%s」", text);
    }
}
```

## 攻撃する
準備はできたので、実際に動作させて攻撃してみます。
起動後にindex.xhtmlにアクセスすると下記の画面が表示されます。

![](/images/post_image_32.png)

「Jump!」ボタンをクリックすると、パラメータが送信されてresult.xhtmlが表示されます。

![](/images/post_image_33.png)

ここで、攻撃者が用意した下記のリンクを踏んだと仮定します。

    http://localhost/jsf-csrf/result.xhtml?text=攻撃

同一ブラウザであれば、同じセッションIDがサーバーに送信されるはずなので、
認証をセッションIDベースで行っている場合は問題なくスルーされるはずです。
というわけで、下記の結果になります。

![](/images/post_image_34.png)

意図していない文字列が登録されてしまいました。

## 対策をする
このように悪意を持ったリンクを踏んでしまうと、
ユーザの意図しない文字列を簡単に登録できてしまうという脆弱性がこのアプリには存在します。
CSRFの対策は、元の画面にトークンを埋め込み、受け取り先でそのトークンの一致を検証することで
正しいルートからのリクエストであることを判定する方法が一般的です。
JSF 2.2からこの仕組みを利用できるようになっているので対策をします。

faces-config.xmlに下記を追記してresult.xhtmlページを保護します。

**faces-config.xml:**

``` xml
<protected-views>
    <url-pattern>/result.xhtml</url-pattern>
</protected-views>
```

JSFでは保護されたページに不正にアクセスすると`javax.faces.application.ProtectedViewException`がスローされます。
そこで、保護されたページにアクセスした場合に表示されるエラーページの設定をweb.xmlに追記します。

**warning.xhtml:**

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<f:view>
    <h1>不正なリクエストです。</h1>
</f:view>
</html>
```

**web.xml:**

``` xml
<error-page>
    <exception-type>javax.faces.application.ProtectedViewException</exception-type>
    <location>/warning.xhtml</location>
</error-page>
```

攻撃者が用意した先ほどのリンクを送信すると下記の結果になりました。

![](/images/post_image_35.png)

うまく機能しているようです。
正しいルートでリクエストを送信すると、下記のようにトークンが付与されていることがわかります。

![](/images/post_image_36.png)

## まとめ
このようにJSFでもCSRF対策が簡単にできるようになりました。
とはいってもそもそもGETでデータを登録しようとしているのが問題のような気もします。
というのは、JSFのManagedBeanのプロパティへバインディングする形でPOSTリクエストを送信している場合、
今回のような攻撃ができないからです。JSFでは画面表示時にコンポーネントツリーを形成していて、
値のバインディングがそのツリーを利用して行われています。
そのため、今回のようにリクエストパラメータを直接プロパティへバインディングしようとしなければ、
簡単に攻撃できないように感じます（色々試してみたのですが攻撃できませんでした。
攻撃方法があるようでしたらこっそり教えて頂けると嬉しいです）


## 参考文献

- [クロスサイトリクエストフォージェリ - Wikipedia](http://ja.wikipedia.org/wiki/クロスサイトリクエストフォージェリ)
- [Java Platform, Enterprise Edition 7: JSON Processing](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/JSF-CSRF-Demo/JSF2.2CsrfDemo.html)
