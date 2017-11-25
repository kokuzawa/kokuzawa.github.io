---
title: "Apache solrで全文検索に触れる"
date: 2014-09-05T00:12:34+09:00
tags: [Solr, Java] 
---
Javaで簡単に全文検索を体験するには Apache Solr (ソーラー)が便利です。  
今回は現時点での最新バージョンである 4.10.0 を利用して全文検索を体験したいと思います。

<!-- MORE -->

## 環境

* OS: Mac OSX 10.9.4
* Java: Oracle Corporation Java HotSpot(TM) 64-Bit Server VM (1.8.0 25.0-b70)
* メモリ: 4GB
* Apache Solr-4.10.0

## Solrを起動する

SolrはWebアプリケーションの形になっています。
ZIPファイルをダウンロードして解凍すると、distフォルダにWARファイルが入っているので、
これをアプリケーションサーバにデプロイすれば良さそうですが、
今回はexampleフォルダにあるstart.jarを単独起動させます。

    java -jar start.jar

起動するとログが出力されて分かるのですが、Jettyを内包しているようです。
数秒で起動するので、下記URLにアクセスして起動していることを確認します。

    http://localhost:8983/solr/

## 文書登録
Solrを紹介しているサイトを確認すると、XMLファイルを読み込ませてインデックスを作成する例が多く見つかります。
ですが、実際の業務としてはOfficeファイルやPDFなどのファイルの内容をインデックス化したいという要求の方が多いので、
XMLではなく、それらのファイルを読み込ませてインデックスを作ってみることにします。
おそらく起動したSolr管理画面からもファイルの登録ができると思うのですが、
今回はJavaで書いたプログラムからファイルの登録を実行します。

JavaからSolrにアクセスするにはApache Solrjというライブラリが必要です。
これはMavenに登録されているのでそこから取得します。

``` xml
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>4.10.0</version>
    <scope>compile</scope>
</dependency>
```

SolrjからPDFファイルを登録するサンプルコードです。

``` java
import org.apache.solr.client.solrj.SolrServer;
import org.apache.solr.client.solrj.impl.HttpSolrServer;
import org.apache.solr.client.solrj.request.AbstractUpdateRequest;
import org.apache.solr.client.solrj.request.ContentStreamUpdateRequest;

import java.io.File;

public class App
{
    public static void main(String... args) throws Exception
    {
        final SolrServer solr = 
                new HttpSolrServer("http://localhost:8983/solr/collection1");
        final ContentStreamUpdateRequest update = 
                new ContentStreamUpdateRequest("/update/extract");
        update.addFile(
                new File("/Users/Katsumi/Downloads/javamagazinevol16.pdf"), 
                "application/pdf");
        update.setParam("literal.id", "javamagazinevol16.pdf");
        update.setAction(AbstractUpdateRequest.ACTION.COMMIT, true, true);
        solr.request(update);
    }
}
```

ちょうど手元に以前ダウンロードした「Javaマガジン日本語版 vol16」があったので、
それを登録しています。実行するとわかるのですがcommons-loggingのクラスがない旨を表すエラーがでます。
Mavenの依存ライブラリには含まれていないようなのですが、内部で使っているのでしょうね。
しょうがないので、commons-loggingも取得するようにMavenに追加します。

``` xml
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
    <scope>runtime</scope>
</dependency>
```

再度実行すると今度はエラーも発生せずに正常に終了しました。
登録したPDFファイルは66ページ、24.6MBですが、数秒で登録が終わりました。

## 文書検索

以下のURLを実行して登録されていることを確認します。

    http://localhost:8983/solr/collection1/select?q=ストリーム&wt=xml

PDFファイル中に「ストリーム」という単語があるので、これを検索条件（q）としています。
また、検索結果フォーマット（wt）はxmlを指定します。他にjsonも指定できるようです。
検索結果は以下のようになりました。

``` xml
<response>
  <lst name="responseHeader">
    <int name="status">0</int>
    <int name="QTime">1</int>
    <lst name="params">
      <str name="q">ストリーム</str>
      <str name="wt">xml</str>
    </lst>
  </lst>
  <result name="response" numFound="1" start="0">
    <doc>
      <arr name="links">...</arr>
      <str name="id">javamagazinevol16.pdf</str>
      <date name="last_modified">2014-05-21T07:44:33Z</date>
      <arr name="title">
        <str/>
      </arr>
      <arr name="content_type">
        <str>application/pdf</str>
      </arr>
      <arr name="content">
        <str>
          JAVA //MARCH/APRIL 2014 / 注目のキーワ ドー： ラムダ式 / Nashorn / 日付と時間 / 組込み
          ...
```

次回はJavaから文書検索をしてみたいと思います。

---
**Change at Sep 6, 2014**  
誤字脱字を修正。
