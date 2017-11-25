---
title: "Derby does not start"
date: 2014-03-09T14:43:26+09:00
tags: [Java, Derby] 
---
Java8を入れたらGlassFishに付属しているDerbyが起動できなくなっていたので、下記を`$JAVA_HONE/jre/lib/security/java.policy`に追記しました。

    permission java.net.SocketPermission "localhost:1527", "listen,resolve";

## 参考
- [Java7u51以降でApache Derbyのネットワークサーバを使う場合の設定](http://d.hatena.ne.jp/seraphy/20140214)
