---
title: "Jenkins用のpom.xmlの基本設定"
date: 2014-05-24T16:30:10+09:00
tags: [Java, Maven, Jenkins] 
---
だんだんと暑くなってきた先週、Java7 を導入できることになって大喜びして
IntelliJ IDEA の Inspect Code を使って Java7 に対応するコードに一括変換をした月曜日、
今年は良い夏を迎えられそうだと思っていたら、OS が Java7 をサポートしてなくてコードを
Java6 にダウングレードするという、モチベーションがだだ下がりな作業したところです。

<!-- MORE -->

ただ Java6 にダウングレードするというものやってられないので、
Eclipse の Web アプリ構成のプロジェクトを Maven 構成のプロジェクトに置き換えようと考えました。
Maven 構成に変えるのはフォルダの位置を変更するだけなのでとても簡単で、
かつ、ライブラリはビルド時に取得してくれるから、今までのように VCS に jar ファイルも一緒にコミットする必要がなくなり、
アプリのリソース自体も小さくなってチェックアウト（クローン）が速くできるようになったりなどメリットが一杯です。

ビルドに ant を使っているので、Jenkins 上でも ant で動作させています。
今度は Maven を使うようにするので、Maven 用に設定を書き換える必要があります。
やりたいことは、ユニットテスト、カバレッジ、PMD、CPD、Findbugs を実行することです。
ant での設定は下記の本を参考にすると簡単なのですが、この本は Maven の設定に詳しくありません。

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?t=moonwhaleblog-22&o=9&p=8&l=as1&asins=4774148911&ref=qf_sp_asin_til&fc1=000000&IS2=1&lt1=_blank&m=amazon&lc1=0000FF&bc1=FFFFFF&bg1=FFFFFF&f=ifr" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

そこで色々なサイトを参考に自分で pom.xml を構築しようとしたのですが、
記述が古かったり、書き方もまちまち、さらには Maven Plugin のバージョンで動作が変わってしまったりなど、
どれを参考にしたら良いのか判断が難しい。
あーでもない、こーでもないとやっているうちに、ユニットテスト、カバレッジ、PMD、CPD、Findbugs を
実行できるようになったので公開したいと思います。

pom.xml の完全なファイルは[こちら](https://bitbucket.org/kokuzawa/pomtemplate)。

## 指定したテストケースだけ実行

``` xml
<build>
    <plugins>
	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-surefire-plugin</artifactId>
	    <version>2.7.2</version>
	    <configuration>
		<systemPropertyVariables>
		    <jdbc.driverClassName>${jdbc.driverClassName}</jdbc.driverClassName>
		    <jdbc.url>${jdbc.url}</jdbc.url>
		    <jdbc.username>${jdbc.username}</jdbc.username>
		    <jdbc.password>${jdbc.password}</jdbc.password>
		</systemPropertyVariables>
		<includes>
		    <include>**/*Test.java</include>
		</includes>
	    </configuration>
	</plugin>
    </plugins>
</build>
```

指定したテストクラスだけを実行するために `maven-surefire-plugin` を利用します。
`systemPropertyVariables` の部分はおまけで、profiles からテスト用の DB 設定を取得しています。
本質は `includes` の部分で、ここに実行したいテストクラスを記述します。

## カバレッジの取得

``` xml
<build>
    <plugins>
	<plugin>
	    <groupId>org.codehaus.mojo</groupId>
	    <artifactId>cobertura-maven-plugin</artifactId>
	    <version>2.5.2</version>
	    <configuration>
		<check/>
		<formats>
		    <format>xml</format>
		</formats>
	    </configuration>
	</plugin>
    </plugins>
</build>
```

カバレッジ取得に `cobertura-maven-plugin` を利用します。
`check` タグは必須なのですが、デフォルトの設定値を変える必要はないため空で宣言しています。
また、`formats` に xml を指定していますが、何も指定していないと site に表示する html が生成されます。
Jenkins で解析するので xml を出力するように指定します。

## PMD の取得

``` xml
<build>
    <plugins>
	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-jxr-plugin</artifactId>
	    <version>2.3</version>
	</plugin>
	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-pmd-plugin</artifactId>
	    <version>3.1</version>
	</plugin>
    </plugins>
</build>

<reporting>
    <plugins>
        <plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-jxr-plugin</artifactId>
	    <version>2.3</version>
	</plugin>
	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-pmd-plugin</artifactId>
	    <version>3.1</version>
	</plugin>
    </plugins>
</repoting>
```

PMD を取得するためのプラグインを指定します。
`maven-jxr-plugin` は PMD 取得に直接関係ないのですが、下記のサイトでの WORNING を回避するために指定しています。

- [PMD plugin: source xref not generated unless JXR runs first](http://maven.40175.n5.nabble.com/PMD-plugin-source-xref-not-generated-unless-JXR-runs-first-td2622198.html)

CPD は PMD と同じ設定で生成することができるので、個別の設定は必要ありません。

## Findbugs の実行

``` xml
<build>
    <plugins>
	<plugin>
	    <groupId>org.codehaus.mojo</groupId>
	    <artifactId>findbugs-maven-plugin</artifactId>
	    <version>2.5.4</version>
	    <configuration>
		<effort>Max</effort>
		<xmlOutput>true</xmlOutput>
	    </configuration>
	</plugin>
    </plugins>
</build>

<reporting>
    <plugins>
	<plugin>
	    <groupId>org.codehaus.mojo</groupId>
	    <artifactId>findbugs-maven-plugin</artifactId>
	    <version>2.5.4</version>
	</plugin>
    </plugins>
</reporting>
```

Findbugs を実行するために `findbugs-maven-plugin` を利用します。
`effort` は分析レベルです。ここでは Max を指定していますが、何も指定しない（Default）でも良いかもしれません。
あとは Jenkins で解析するので xml での出力が必要です。

## その他
Maven-3.2.1 で実行しようとすると下記サイトでのエラーが出ることがあります。

- [Maven 3.2.1: IllegalAccessError on AbstractMapBasedMultimap](https://issues.jenkins-ci.org/browse/JENKINS-22252)

理由はコメントに書いてある通りなのですが、Jenkins で利用している Guava のバージョンと Maven の Guava のバージョンが
一致していないために発生する問題のようです。なので、最新の Maven ではなく一つ古い 3.2.1 を使う必要があるかもしれません。

ちなみに、Jenkins での Maven の goal は下記を指定しています。
（cobertura でテストクラスのコンパイルも行われるのですが、Java6 だと JDK のバグでコンパイラが止まってしまうので、
あえて `compile` を呼び出しています）

`clean compile cobertura:cobertura pmd:pmd pmd:cpd findbugs:findbugs javadoc:javadoc`

---
**Change at May 29, 2014**  
resource ゴールが呼ばれないため、Jenkinsで指定している goal のうち、compiler:compile compiler:testCompile を compile に変更。

