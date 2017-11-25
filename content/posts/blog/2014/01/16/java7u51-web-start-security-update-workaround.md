---
title: "java7u51 Web Startセキュリティアップデートの回避方法"
date: 2014-01-16T07:08:26+09:00
tags: ["Java", "Web Start"] 
---
1月14日にJava7u51がリリースされました。
このアップデートによってセキュリティが強化されて、署名のないアプレットが起動できなくなりました。
また署名があっても、ユーザ独自の署名の場合ではその証明書をクライアントにインポートしておかないと、
同じく起動することができないようです。

あまり頻繁に行う作業ではないので、手順をまとめておきたいと思います。

## 1. MANIFEST.MFの編集

jarファイル内のMANIFEST.MFに下記を追記します。

    Permissions: all-permissions

Mavenを利用している場合は、下記をpom.xmlに追加することで、
ビルド時にMANIFEST.MFに追加することができます。

``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.4</version>
            <configuration>
                <archive>
                    <manifestEntries>
                        <Permissions>all-permissions</Permissions>
                        <Codebase>*</Codebase>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Antであれば下記で同様のことができます。

``` xml
<target name="dist">
    <jar destfile="${basedir}/out/sample.jar" basedir="${basedir}/out/classes/">
        <manifest>
            <attribute name="Permissions" value="all-permissions"/>
            <attribute name="Codebase" value="*"/>
        </manifest>
    </jar>
</target>
```

## 2. jnlpファイルの編集

jnlpファイルに下記を追記します。

    <security>
        <all-permissions/>
    </security>

## 3. keystoreの生成

keystoreを生成します。[]内は任意の文字列を指定します。

    keytool -selfcert -alias [sample] -genkey -keystore [sample.keystore] -validity [3650]

## 4. 証明書の生成

    keytool -export -keystore [sample.keystore] -alias [sample] -file [sample.csr]

## 5. jarファイルの署名

    jarsigner -keystore [sample.keystore] [sample.jar] [sample]

## 6. 証明書のインポート

証明書をインポートすることでAppletを起動できるようになります。  
Windowsの場合は、「コントロールパネル」 -> 「Java」 -> 「セキュリティタブ」 -> 「証明書の管理」と選択、
ユーザタブで証明書タイプを「署名者のCA」を選び、sample.csrをインポートします。

これでWeb StartでAppletを起動すると、
警告ダイアログが表示されるようになります。

Enjoy !

