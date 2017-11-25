---
title: "JdbcRealm with WildFly 9.0.1.Final"
date: 2015-12-25T20:50:08+09:00
tags: [Java, WildFly, JavaEE] 
---
以前、「[WildFlyでJdbcRealm](http://kokuzawa.github.io/blog/2014/08/23/wildflydejdbcrealmwoshe-ding-suru/)」 という記事を書きました。
これを現在インストールしている9.0.1.Final上で設定したところ、認証がうまく行われないことがわかりました。
大枠の変更はないのですが、DBに登録するパスワードのハッシュ文字列が当時とは異なる値である必要があったので、
忘れないようにメモしておきます。

<!-- MORE -->

## 差分

WildFly 8.0.0.Finalの時の設定:

```xml
<security-domain name="app" cache-type="default">
    <authentication>
        <login-module name="app_auth" code="Database" flag="required">
            <module-option name="dsJndiName" value="java:jboss/datasources/ExampleDS"/>
            <module-option name="principalsQuery" value="SELECT PASSWORD FROM ACCOUNTS WHERE EMAIL = ?"/>
            <module-option name="rolesQuery" value="SELECT r.ROLENAME, 'Roles' FROM ROLES r, ACCOUNTS a WHERE r.ACCOUNTID = a.ACCOUNTID AND a.EMAIL = ?"/>
            <module-option name="hashAlgorithm" value="SHA-256"/>
            <module-option name="hashEncoding" value="HEX"/>
        </login-module>
    </authentication>
</security-domain>
```

WildFly 9.0.1.Finalの設定:

```xml
<security-domain name="app" cache-type="default">
    <authentication>
        <login-module name="app_auth" code="Database" flag="required">
            <module-option name="dsJndiName" value="java:jboss/datasources/ExampleDS"/>
            <module-option name="principalsQuery" value="SELECT PASSWORD FROM ACCOUNTS WHERE EMAIL = ?"/>
            <module-option name="rolesQuery" value="SELECT r.ROLENAME, 'Roles' FROM ROLES r, ACCOUNTS a WHERE r.ACCOUNTID = a.ACCOUNTID AND a.EMAIL = ?"/>
            <module-option name="hashAlgorithm" value="SHA-256"/>
            <module-option name="hashEncoding" value="base64"/>
        </login-module>
    </authentication>
</security-domain>
```

違いは module-option の hashEncoding の値。
8.0.0.Finalの時は`HEX`であり、9.0.1.Finalでは`base64`にしています。
これはパスワードのハッシュエンコーディングの形式を指定している部分なのですが、
9.0.1.FinalではHEXを認識していない模様。
なので、DBに登録するパスワードのハッシュ文字列も設定に合わせて`HEX`から`base64`に変更します。

## パスワードのハッシュ文字列生成方法

WildFlyには`base64`のハッシュ文字列を生成するモジュールが入っているようです。
以下のコマンドで指定文字列のBase64ハッシュ値を取得することができます。

    java -cp $JBOSS_HOME/modules/system/layers/base/org/picketbox/main/picketbox-4.9.2.Final.jar org.jboss.security.Base64Encoder [任意文字列] SHA-256

## 参考サイト

- [JDBC Realm and Form Based Authentication with WildFly 8.2.0.Final, Primefaces 5.1 and MySQL 5](http://blog.eisele.net/2015/01/jdbc-realm-wildfly820-primefaces51.html)

