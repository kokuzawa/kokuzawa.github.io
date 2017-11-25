---
title: "JAX-RSで複数ファイルをアップロードするには"
date: 2013-12-22T09:20:55+09:00
tags: [Java, JavaEE] 
---
この記事は[Java Advent Calendar 2013](http://www.adventar.org/calendars/145)の22日目の記事です。  
昨日は[@nagaseyasuhito](https://twitter.com/nagaseyasuhito)さんの「[mvn siteのtips三連発](http://www.nagaseyasuhito.net/2013/12/21/368/)」でした。  
明日は monzou さんです。

ファイルアップロードを実現するために何を使っていますか？  
私はもっぱらJAX-RSを使っています。  

ファイルアップロードはAX-RSの仕様には含まれていないのですが、ほとんどの実装でサポートされているようです。
その実装の中から、今回はJerseyを使った複数ファイルのアップロードについて紹介しようかと思います。  

今回利用したJerseyはGlassFish4に含まれているものを利用します。  
ライブラリとしては以下のjarになります。

``` xml
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-multipart</artifactId>
    <version>2.4.1</version>
</dependency>
```

さっそくHTMLから。複数のファイルをアップロードするので同じnameのfileフィールドを用意します。  

``` html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>JAX-RS upload multiple files example</title>
</head>
<body>
    <form enctype="multipart/form-data" method="post" action="rest/upload/multipleFiles">
        <div><input type="file" name="file"/></div>
        <div><input type="file" name="file"/></div>
        <input type="submit" value="Upload">
    </form>
</body>
</html>
```

で、実際にリクエストを受け取るJavaのコード。

``` java
package org.katsumi.resources;

import org.glassfish.jersey.media.multipart.FormDataBodyPart;
import org.glassfish.jersey.media.multipart.FormDataParam;

import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

/**
 * 複数ファイルのアップロードサンプル
 * @author Katsumi
 * @since 2013/12/21
 */
@Path("/upload")
public class Upload
{
    @POST
    @Consumes(MediaType.MULTIPART_FORM_DATA)
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/multipleFiles")
    public String upload(@FormDataParam("file")List<FormDataBodyPart> bodyParts) throws IOException
    {
        for (FormDataBodyPart part : bodyParts) {
            Files.copy(part.getValueAs(InputStream.class),
                    Paths.get("/Users/Katsumi/Downloads", part.getFormDataContentDisposition().getFileName()));
        }
        return "finish!!";
    }
}
```

fileフィールドの値は、@FormDataParam("file")アノテーションを引数につけることで受け取ることができます。
また、今回は複数のファイルを受け取るので、引数の方を`java.util.List`にしています。
`FormDataBodyPart`に実際のファイルの情報が含まれているので、ここから必要なものを取り出します。

ネットで検索すると、ファイルのアップロードを受け取る際に、
`InputStream`と`FormDataContentDisposition`の2つをパラメータとして受け取るサンプルが良く見つかりますが、
`FormDataBodyPart`にはこの2つの情報が含まれているので、こちらを利用した方が良いのではないかと思います。

ちなみに、`FormDataBodyPart`から`InputStream`を取り出すには`getValueAs(InputStream.class)`を利用します。  
ここがちょっと分かりにくいところかもしれません。

あと、このコードだと、日本語ファイル名の場合に文字化けしてしまうので適切なエンコードが必要になりますが、
文字コードの変換部分は今回の本質的な部分ではないので割愛します。

もう一点、今回の実装で注意する部分があります。  
JAX-RSのリソースクラスをweb.xmlで登録しているのですが、マルチパートを処理するに当たり登録対象として
`org.glassfish.jersey.media.multipart.MultiPartFeature`クラスを含める必要あるようです。
ですので、web.xmlの該当部分は下記のような宣言としています。

``` xml
<servlet>
    <servlet-name>MyApplication</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>jersey.config.server.provider.classnames</param-name>
        <param-value>
            org.katsumi.resources.Upload,
            org.glassfish.jersey.media.multipart.MultiPartFeature
        </param-value>
    </init-param>
</servlet>
```

完全なコードは[Bitbacket](https://bitbucket.org/kokuzawa/jaxrs-rs-upload-multiple-files/src)にアップしています。

## ちょっと発展
HTML5ではファイルインプットフィールドにmultiple属性が追加されました。  
これを使うと、一つのファイルフィールドで複数のファイルを設定できるようになります。  

HTMLをmultipleに変更します。

``` html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <title>JAX-RS upload multiple files example</title>
</head>
<body>
    <form enctype="multipart/form-data" method="post" 
          action="rest/upload/multipleFiles">
        <div><input type="file" name="file" multiple="multiple"/></div>
        <input type="submit" value="Upload">
    </form>
</body>
</html>
```

ファイルを複数選択します。

![](/images/post_image_27.png)

Uploadボタンをクリックするとファイルが無事アップロードできました。
Javaのコードの修正は必要ないですね。当たり前と言えば当たり前なのですが... :P

ファイルアップロードというと面倒な処理を書かないといけないと思うかもしれませんが、
JAX-RSというかJerseyを利用すると、このように簡単にアップロードを行うことができます。

皆さんも試してみてはいかがでしょうか？

Enjoy!

