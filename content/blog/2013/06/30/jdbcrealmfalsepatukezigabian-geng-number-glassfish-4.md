---
title: "JDBCRealmのパッケージが変更 #glassfish-4"
date: 2013-06-30T11:09:00+09:00
tags: [Java, GlassFish] 
---
GlassFish-4.0からJDBCRealmクラスのパッケージが変更になった。  
今までのパッケージは下記の通り。

``` java
com.sun.enterprise.security.auth.realm.jdbc.JDBCRealm
```

GlassFish-4.0からは下記のパッケージとなる。

``` java
com.sun.enterprise.security.ee.auth.realm.jdbc.JDBCRealm
```

Embeddedとして利用しているとかじゃないとあまり影響はないかもしれない。
