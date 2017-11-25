---
title: "Convert the Alerts component of 'Bootstrap' to a component of JSF 2.0"
date: 2013-01-13T18:46:00+09:00
tags: [Java, JavaEE, JSF, Bootstrap] 
---

JSFのMessagesコンポーネントは、Managed Beanで設定されたFacesMessageを出力する為のコンポーネントです。
出力方法は、listとTableの二つのレイアウトを利用した方法があり、デフォルトはlistレイアウトです。
listレイアウトは以下のような出力になります。  
(Bootstrapの`alert alert-error`を適用)

![](/images/post_image_1.png)

これに対してTableレイアウトは以下のような出力になります。

![](/images/post_image_2.png)

どちらもエラーを表示するには十分ですが、ユーザとしては確認したらエラー情報を消したいところです。
MessagesコンポーネントはHTMLをカスタマイズする事ができません。
正確にはレンダラをカスタマイズすれば、出力するHTMLを書き換える事が出来ますし、
そういった情報を扱ったブログもありますが、Messagesレンダラのカスタマイズは、
com.sunパッケージのクラスを継承して拡張する必要があり、JSFの実装依存のコードになってしまいます。
もちろん独自で一からレンダラを書いても良いのですが、あまり現実的ではありません。
そこで、標準のMessagesコンポーネントではなくBootstrapのAlertsコンポーネントを使うことにします。

BootstrapのAlertsコンポーネントはクローズボタンを表示する事ができます。
このクローズボタンをクリックする事により、エラー情報を消すことができます。
クローズボタンを出すには、HTMLを下記のように記載する必要があります。

``` html
<div class="alert alert-error">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    <h4>Summary Message</h4>
    Detail Message
</div>
```

これをJSF合成コンポーネントにします。
webapp/resources/bootstrapフォルダを作成し、alert.xhtmlファイルを配置します。(フォルダ構成はMavenです)

**alert.xhtml:**

``` html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
      "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:cc="http://java.sun.com/jsf/composite"
      xmlns:c="http://java.sun.com/jsp/jstl/core">

    <!-- INTERFACE -->
    <cc:interface>
    </cc:interface>

    <!-- IMPLEMENTATION -->
    <cc:implementation>
        <c:forEach var="message" items="#{facesContext.messageList}">
            <c:if test="#{message.severity == 'ERROR 2'}">
                <div class="alert alert-error">
                    <button type="button" class="close" 
                            data-dismiss="alert">&times;</button>
                    <h4>#{message.summary}</h4>
                    #{message.detail}
                </div>
            </c:if>
        </c:forEach>
    </cc:implementation>
</html>
```

JSF合成コンポーネントはinterfaceの部分にコンポーネントの属性を、implementationの部分に実装を書きます。
今回は属性は不要なので、実装部分だけになります。
FacesMessageはFacesContext.messageListから取得します。
コンポーネントはエラーメッセージが設定されている場合のみ表示します。
エラーメッセージはFacesMessage.severityがFacesMessage.SEVERITY_ERRORのものになりますが、
EL式ではStatic fieldを比較値として比較できません。
そのため、FacesMessage.SEVERITY_ERRORの文字列表現である'ERROR 2'と比較し、一致するものをエラーメッセージとして判定します。
また、複数設定されている場合を考慮して、forEachを利用し、FacesMessageの数だけAlertsコンポーネントを表示します。

配置したJSF合成コンポーネントは、http://java.sun.com/jsf/composite/bootstrap というネームスペースで利用できます。
http://java.sun.com/jsf/composite/ にフォルダ名であるbootstrapを付けるだけです。

``` html
<html xmlns:bs="http://java.sun.com/jsf/composite/bootstrap">
    <bs:alert/>
</html>
```

実際に表示すると以下のようになります。

![](/images/post_image_3.png)

エラーメッセージの右上にクローズボタンが表示され、クリックする事によりメッセージを閉じることができるようになりました。

