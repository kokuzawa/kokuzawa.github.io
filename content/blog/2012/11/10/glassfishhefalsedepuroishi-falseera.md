---
title: "GlassFish3.1.2へのデプロイ時のエラー"
date: 2012-11-10T07:44:00+09:00
tags: [Java, GlassFish]
---

> java.util.concurrent.ExecutionException: com.sun.faces.config.ConfigurationException: Unable to parse document 'bundle://213.0:1/com/sun/faces/jsf-ri-runtime.xml': DTD factory class org.apache.xerces.impl.dv.dtd.DTDDVFactoryImpl does not extend from DTDDVFactory.

JSFを利用していると発生しているようなので、warファイル内のjavax.faces-2.1.4.jarを取り除いて再度デプロイ。でも変化なし。
（JSFのライブラリはGlassFishに含まれている。バージョンはわからないけど）

何度か再デプロイをしていると起動できるので、それほど真剣に調べていなかったけど、本腰をいれて調べてみることにした。

### 結果
ログを見ると`org.apache.xerces.impl.dv.dtd.DTDDVFactoryImpl`関係でエラーが発生して`bundle://213.0:1/com/sun/faces/jsf-ri-runtime.xml`がパースできないことがわかる。

っていうか`bundle://213.0:1/com/sun/faces/jsf-ri-runtime.xml`って何だろう？ひとまずこれについてはあとで調査。

おそらくGlassFishで利用しているXMLパーサとぶつかっているせいだと思う。warファイル内からxercesImpl-2.8.1.jarを取り除いてデプロイしたところ正常に起動ができた。
取り除いても正常に起動できたのでXMLパーサがGlassFishに含まれてるのは間違いないと推測できるが、GlassFishのどこにXMLパーサが含まれているのかがわからない。

#### 追記: Nov 11, 2012
運用環境はTomcatだからxercesImpl-2.8.1.jarを取り除いちゃダメなんじゃないかと思う。
