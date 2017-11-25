---
title: "WildFlyでJdbcRealm"
date: 2014-08-23T15:50:18+09:00
tags: [Java, JavaEE, WildFly] 
---
今回は WildFly 8.0.0.Final を利用してJdbcRealmを試してみます。

<!-- MORE -->

## WildFly の設定

WildFly 8.0.0.Final に JDBCRealm を構築します。  
まず、PostgreSQL を使ってとてもシンプルなテーブル構成を作ります。

![](/images/post_image_28.png)

Security Domain を追加します。
追加は WildFly の GUI コンソールから行います。
追加する Security Domain は Name: app, Cache Type: default です。

![](/images/post_image_29.png)

追加した Security Domain を開き、Login Module を追加します。
追加する Login Modile は Code: Database, Flg: required です。

![](/images/post_image_30.png)

追加した Login Module に Module Option を追加します。

![](/images/post_image_31.png)

追加する Module Option は下記表になります。
dsJndiName で指定するのは事前に登録した Datasource です。

 Key             | Value               
:----------------|:----------------------------------------------------------------------------------------------------
 dsJndiName      | java:/jdbc/realmSample 
 hashAlgorithm   | SHA-256 
 hashEncoding    | HEX
 principalsQuery | SELECT password FROM accounts WHERE email = ?
 rolesQuery      | SELECT r.rolename, 'Roles' FROM roles r, accounts a WHERE r.accountid = a.accountid AND a.email = ?

## データの投入

最初に作ったテーブルにデータを投入します。
パスワードに設定するのは、SHA256で暗号化、HEXエンコードした文字列です。
ここでは 'test' という文字列をパスワードにしています。

``` sql
INSERT INTO accounts (email, password) VALUES ('hoge', '9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08');
INSERT INTO roles (rolename, accountid) VALUES ('MEMBER', 1);
```

## jboss-web.xml の設定

追加した Security Domain を利用するために、jboss-web.xml へ Security Domain を指定します。

``` xml
<security-domain>app</security-domain>
```

## web.xml の設定

BASIC認証が行われるように web.xml を設定します。

``` xml
<security-role>
    <role-name>MEMBER</role-name>
</security-role>

<security-constraint>
    <web-resource-collection>
        <web-resource-name>Member Resource</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>MEMBER</role-name>
    </auth-constraint>
</security-constraint>

<login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>Authentication</realm-name>
</login-config>
```

これですべての設定が完了です。
WildFly にアプリをデプロイしてアクセスするとBASIC認証のダイアログが表示されると思います。
そこで事前に登録したアカウント情報を入力すると認証に成功するはずです。

---
- Oct 28, 2014 脱字修正
