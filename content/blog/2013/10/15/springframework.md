---
title: "springframeworkのjarファイル一式をダウンロードするには？"
date: 2013-10-15T00:16:00+09:00
tags: [Java, Spring, Maven]
---
久しぶりにSpringのサイトを見に行ったら、Spring Framework一式がZIPでダウンロードできなくなってた。
サイトは今風になって素敵なんだけど。

MavenとかGradleを使って取得するのが正しい方法として挙げられているけれども、一式を取得する方法は特に記載がない。
Mavenを利用したことがなくて一式欲しいという需要があるようなので、ここで一つの方法を書いてみようと思う。
Gradleの方が記述が簡単そうだけど、ちゃんと使ったことがないので今回はMavenを利用する。

Mavenでは maven-asembly-plugin がこの要望を満たしてくれる。
maven-assembly-plugin は pom.xml とは別に個別の設定ファイルが必要なので、それを用意する。

**distribution.xml:**

``` xml
<?xml version="1.0"?>
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2
                              http://maven.apache.org/xsd/assembly-1.1.2.xsd">
    <id>distribution</id>
    <formats>
        <format>zip</format>
    </formats>
    <dependencySets>
        <dependencyset>
            <unpack>false</unpack>
            <scope>runtime</scope>
            <outputDirectory>/out</outputDirectory>
        </dependencyset>
    </dependencySets>
</assembly>
```

ファイルの内容を簡単に説明すると、format には出力形式として zip, tar, tar.gz が指定できる。
unpack では、取得する jar ファイルを解凍して一つの jar とするかどうかを指定する。
scope は、Maven の dependency の scope と同じ意味なので、必要な値を指定、
outputDirectory は zip の出力先を指定する。

次にpom.xmlにプラグインの追加とdistribution.xmlの場所を定義する必要がある。

**pom.xml:**

``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2.1</version>
            <configuration>
                <descriptors>
                    <descriptor>distributions.xml</descriptor>
                </descriptors>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Spring Framework のライブラリはリポジトリを別途指定する必要があるので、それを pom.xml に定義する。

**pom.xml:**

``` xml
<repositories>
    <repository>
        <id>com.springsource.repository.bundles.release</id>
        <name>SpringSource Enterprise Bundle Repository - SpringSource Bundle Releases</name>
        <url>http://repository.springsource.com/maven/bundles/release</url>
    </repository>
    <repository>
        <id>com.springsource.repository.bundle.external</id>
        <name>SpringSource Enterprise Bundle Repository - External Bundle Releases</name>
        <url>http://repository.springsource.com/maven/bundles/external</url>
    </repository>
    <repository>
        <id>com.springsource.repository.libraries.release</id>
        <name>SpringSource Enterprise Bundle Repository - SpringSource Library Releases</name>
        <url>http://repository.springsource.com/maven/libraries/release</url>
    </repository>
    <repository>
        <id>com.springsource.repository.libraries.external</id>
        <name>SpringSource Enterprise Bundle Repository - External Library Releases</name>
        <url>http://repository.springsource.com/maven/libraries/external</url>
    </repository>
</repositories>
```

あとは Spring framework の実体を指す dependency を追加すれば完成。
これは[ここ](http://ebr.springsource.com/repository/app/library/version/detail?name=org.springframework.spring&version=3.2.3.RELEASE)を
参考にすれば良いと思う。

**pom.xml:**

``` xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>org.springframework.spring-library</artifactId>
    <type>libd</type>
    <version>3.2.3.RELEASE</version>
</dependency>
```

これでpom.xmlが完成したので、あとは以下のコマンドを実行すればdistribution.xmlで定義したoutディレクトリに
Spring Frameworkのjarファイル一式が含まれたzipファイルが生成されるはず。

    $ mvn assembly:assembly

Best regards.

