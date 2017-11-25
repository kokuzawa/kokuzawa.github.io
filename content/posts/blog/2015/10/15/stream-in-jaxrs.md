---
title: "JAX-RSでStreamを扱う"
date: 2015-10-15T02:44:04+09:00
tags: [Java, JAX-RS] 
---
JAX-RSでExcelファイルをダウンロードする際にストリームを直接触る必要があって、
どうすればストリームにアクセスできるかちょっと調べてみました。
Excelファイルの生成にはApache POIを使っています。

<!-- MORE -->

Apache POIでExcelファイルを生成する場合、下記のようなコードを書きます。

``` java
final Workbook wb = new HSSFWorkbook();
final FileOutputStream fileOut = new FileOutputStream("workbook.xls");
wb.write(fileOut);
fileOut.close();
```

生成したファイルをファイルとして保存せずにServletでダウンロードしようとした場合、
そのコードは下記のように`HttpServletResponse#getOutputStream()`でアウトプットストリームを取得し、
レスポンスボディに対してストリーミング処理をすることになります。

``` java
final Workbook wb = new HSSFWorkbook();
final FileOutputStream fileOut = new FileOutputStream(response.getOutputStream());
wb.write(fileOut);
fileOut.close();
```

ここからが本題です。  
JAX-RSでファイルをダウンロードするにはどうしたら良いのか。  
通常のファイルの場合は下記のようなコードを書くことで実現できます。

``` java
final File file = new File("workbook.xls");
return Response.ok(file).build();
```

Servletでのダウンロードのように、
レスポンスボディに対してストリーミング処理をする場合は`javax.ws.rs.core.StreamingOutput`クラスを利用します。
そのコードは下記のようになります。

``` java
final Workbook wb = new HSSFWorkbook();
final StreamingOutput so = out -> wb.write(out);
return Response.ok(stream).build();
```

