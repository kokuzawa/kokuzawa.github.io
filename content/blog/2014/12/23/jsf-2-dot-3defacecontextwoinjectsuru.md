---
title: "[JSF-2.3]FacesContextをInjectする"
date: 2014-12-23T15:02:06+09:00
tags: [Java, JSF, WildFly] 
---
JSF-2.3の新しい機能として、UIViewRoot, ViewMap, ApplicationMap, ExternalContext, FacesContextがInjectできるようになります。
この記事ではFacesContextの古い取得方法である`FacesContext.getCurrentInstance()`と、
Injectを利用した取得方法のサンプルを記載します。

<!-- MORE -->

サンプルでは画面のボタンをクリックすると、`<h:messages/>`タグと`FacesContext.addMessage(String, FacesMessage)`を利用して
同じ画面上にインフォメーションメッセージを表示します。

最初にFacesContextの古い取得方法を利用したManaged Beanです。

``` java
import javax.enterprise.context.RequestScoped;
import javax.faces.application.FacesMessage;
import javax.faces.context.FacesContext;
import javax.inject.Inject;
import javax.inject.Named;

@Named
@RequestScoped
public class IndexBean
{
    public void doClick()
    {
        final FacesContext context = FacesContext.getCurrentInstance();
        context.addMessage(null, new FacesMessage(FacesMessage.SEVERITY_INFO, "summary", "detail"));
    }
}
```

次にInjectを利用してFacesContextを取得するManaged Beanです。

``` java
import javax.enterprise.context.RequestScoped;
import javax.faces.application.FacesMessage;
import javax.faces.context.FacesContext;
import javax.inject.Inject;
import javax.inject.Named;

@Named
@RequestScoped
public class IndexBean
{
    @Inject
    private FacesContext context;

    public void doClick()
    {
        context.addMessage(null, new FacesMessage(FacesMessage.SEVERITY_INFO, "summary", "detail"));
    }
}
```

画面に表示するXHTMLは下記のようにしています。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html
        PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html">

<h:head>
    <title>Simple JSF Facelets page</title>
</h:head>

<h:body>
    <h:form>
        <h:messages showDetail="true"/>
        <h:commandButton actionListener="#{indexBean.doClick}" value="OK"/>
    </h:form>
</h:body>

</html>
```

このアプリをWildFly-8.0.0.Finalにデプロイします。
WildFlyにはJSFの実装が含まれているので、JSF-2.3が含まれるアプリをデプロイすると起動時にエラーになってしまいます。
そこで、アプリに含まれるJSF実装を利用させるために、下記の設定をweb.xmlに追加します。
パラメータを追加するだけで利用する実装を切り替えてくれるので便利ですね。

``` xml
<context-param>
    <param-name>org.jboss.jbossfaces.WAR_BUNDLES_JSF_IMPL</param-name>
    <param-value>true</param-value>
</context-param>
```


