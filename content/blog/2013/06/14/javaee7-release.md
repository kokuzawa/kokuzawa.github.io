---
title: "JavaEE 7 リリース！"
date: 2013-06-14T01:53:00+09:00
tags: [Java, JavaEE] 
---
JavaEE 7 がというかGlassFish 4 がリリースされた。  
会社でテストサーバとして使っているのは、GlassFish 3.1.2.2 なので早速入れ替えてみた。
余談だが運用サーバはTomcatなので、どちらかというとテストサーバの方が性能が良い。

で、意気揚々と起動してみた訳なのだが、起動できない。  
理由は単純で、GlassFish 4 はJDK7が必須にもかかわらず、プロジェクトのJDKのバージョンがJDK6なのである。
まあコンパイル時にサーバのクラスのバージョンよりも古いバージョンでコンパイルしているよ、という警告が出ていたので
何となくは思っていたのだが、改めて起動できないとちょっとショックではある。

ひとまず、DataSourceの設定が今までと同じように出来ることは確認したので、
それについて書こうと思う。

<!-- MORE -->

## ドライバの配置
GlassFishで利用できるJDBCドライバには下記のものがある。

**[Administration Guide](https://glassfish.java.net/docs/4.0/administration-guide.pdf):**

> ■ IBM DB2 Database Type 2 Driver  
> ■ IBM DB2 Database Type 4 Driver  
> ■ Java DB/Derby Type 4 Driver  
> ■ MySQL Server Database Type 4 Driver  
> ■ Oracle 10 Database Driver  
> ■ Oracle 11 Database Driver  
> ■ PostgreSQL Type 4 Driver  
> ■ DataDirect Type 4 Driver for IBM DB2 Database  
> ■ DataDirect Type 4 Driver for IBM Informix  
> ■ DataDirect Type 4 Driver for Microsoft SQL Server Database  
> ■ DataDirect Type 4 Driver for MySQL Server Database  
> ■ DataDirect Type 4 Driver for Oracle 11 Database  
> ■ DataDirect Type 4 Driver for Sybase Database  
> ■ Inet Oraxo Driver for Oracle Database  
> ■ Inet Merlia Driver for Microsoft SQL Server Database  
> ■ Inet Sybelux Driver for Sybase Database  
> ■ JConnect Type 4 Driver for Sybase ASE 12.5 Database

沢山あるので困らないと思う。主に使っているのはPostgreSQLなので、今回はそのドライバを使うことにした。
ドライバといってもjarファイルなので、ダウンロードしてきたjarファイルをGlassFishが認識できるクラスパスに配置する必要がある。
これもAdministration Guideに記載があり、下記にあるように`domain-dir/lib`に配置すれば良いようだ。以前との変更点は無い。

**[Administration Guide](https://glassfish.java.net/docs/4.0/administration-guide.pdf):**

> Making the JDBC Driver JAR Files Accessible
> To integrate the JDBC driver into a GlassFish Server domain, copy the JAR files into the domain-dir/lib directory, then restart the server.

サーバを起動したら`localhost:4848`にアクセスし、Resources -> JDBC -> JDBC Connection Pools と選択する。

![](/images/post_image_11.png)

Newボタンをクリックすると、コネクションプールの作成画面に変わる。
今回はPostgreSQLのDataSourceを作るので、Resource Typeは`javax.sql.DataSource`、Database Driver Vendorは`Postgresql`を選択する。

![](/images/post_image_12.png)

必要事項の入力が終わったらNextボタンをクリックしよう。
すると、コネクションプールの詳細な設定画面が表示されるが、上の方の設定はひとまず変更せず、
一番下にあるAdditional Propertiesを設定する。

![](/images/post_image_13.png)

設定が終わったらFinishボタンをクリックしよう。っとその前に上の方の設定にPingというのがあるので、
これをONにしておく。ここをONにしておけば、コネクションプールがちゃんとDBと接続できるか確認してくれるのだ。
うまく接続できれば画面上部に成功を表すメッセージが表示されるはず。

次にResources -> JDBC -> JDBC Resources と選択する。

![](/images/post_image_14.png)

Newボタンをクリックしてリソース設定画面を表示する。
Pool Nameにはさっき作成したコネクションプールを指定する。あとはJNDI Nameに任意の名前を設定してOKボタンをクリックすればJNDIリソースの完成だ。
このリソースは、こんな風に取得することができる。

``` java
final Context ctx = new InitialContext();
final DataSource dataSource = (DataSource) ctx.lookup("jdbc/demo");
```

ここまで設定できれば、あとは色々なWebアプリから利用するだけ。
なんだかJavaEE 7 にはあまり関係がないが、今までと同じように設定が出来るということを報告しておく。
 

