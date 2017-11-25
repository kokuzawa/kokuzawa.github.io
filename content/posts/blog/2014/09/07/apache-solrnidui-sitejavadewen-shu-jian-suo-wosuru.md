---
title: "Javaで文書検索をする (Apache Solr)"
date: 2014-09-07T22:04:16+09:00
tags: [Java, Solr] 
---
[前回](http://kokuzawa.github.io/blog/2014/09/05/apache-solrdequan-wen-jian-suo-nihong-reru/)
Apache Solrに対してJavaで文書登録をして、APIでその結果が取得できるところまでやってみました。
今回はAPIではなく、登録した文書をJavaで検索してみようと思います。

<!-- MORE -->

## 環境

* OS: Mac OSX 10.9.4
* Java: Oracle Corporation Java HotSpot™ 64-Bit Server VM (1.8.0 25.0-b70)
* メモリ: 4GB
* Apache Solr-4.10.0

## Javaで文書検索

文書登録時と同じく、Apache Solrjが必要です。
ライブラリの取得方法は前回を参照して頂くとして、早速サンプルコードです。

``` java
import org.apache.solr.client.solrj.SolrQuery;
import org.apache.solr.client.solrj.SolrServer;
import org.apache.solr.client.solrj.impl.HttpSolrServer;
import org.apache.solr.client.solrj.response.QueryResponse;
import org.apache.solr.common.SolrDocument;
import org.apache.solr.common.SolrDocumentList;

public class SolrClient
{
    public static void main(String... args) throws Exception
    {
        final SolrServer solr = new HttpSolrServer("http://localhost:8983/solr/collection1");
        final SolrQuery solrQuery = new SolrQuery();
        solrQuery.set("q", "ストリーム");
        final QueryResponse response = solr.query(solrQuery);
        final SolrDocumentList results = response.getResults();
        final SolrDocument document = results.get(0);
        System.out.println(document.get("id"));
        System.out.println(document.get("content_type"));
    }
}
```

条件は前回と同じ「ストリーム」を指定します。
上記を実行すると以下のような出力結果になります。

    javamagazinevol16.pdf
    [application/pdf]

## サポートしているファイルタイプ

PDFファイルの登録と検索ができることは確認しましたが、他のファイルはどうなのか気になります。
業務アプリとして利用するにはOfficeファイル、みんなが大好きExcelは検索したいところです。
Solrのリファレンスガイドを読むと下記のファイルをサポートしているようです。

    xml, json, csv, pdf, doc, docx, ppt, pptx, xls, xlsx, odt, odp, ods, rtf, htm, html

業務で必要となりそうなファイルは大体そろっていますが個人的にはiWorkの文書もサポートして欲しいところ...。
内部的にはApache Tikaを利用しているので、iWorkもサポートされているはずと思い試してみたのですが、
下記エラーが出て登録ができませんでした。

``` java
Exception in thread "main" org.apache.solr.client.solrj.SolrServerException: error reading streams
	at org.apache.solr.client.solrj.impl.HttpSolrServer.createMethod(HttpSolrServer.java:434)
	at org.apache.solr.client.solrj.impl.HttpSolrServer.request(HttpSolrServer.java:210)
	at org.apache.solr.client.solrj.impl.HttpSolrServer.request(HttpSolrServer.java:206)
```

何か方法があるのかもしれませんが、今日のところは時間切れ。
あとでまた調べてみたいと思います。


