---
title: "PrimeNGを組み込む"
date: 2016-12-28T11:02:35+09:00
tags: [Angular2]
---
[昨日](http://kokuzawa.github.io/blog/2016/12/28/intellij-ideaangular2/)、 Angular2 アプリを IntelliJ IDEA で動かすところまでやってみました。
今日は本来の目的である PrimeNG を使ってみます。

<!-- more -->

## PrimeNG とは

JSF の実装の一つである PrimeFaces と同じところが作っている Angular2 用の Rich UI Components です。
PrimeFaces とほぼ同じ（まだ PrimeNG の方が少ない）コンポーネント群を Angular2 から利用できるプロダクトです。
現在の最新バージョンは 1.1.2 です。

## PrimeNG のインストール

PrimeNG の [Setup](http://www.primefaces.org/primeng/#/setup) に書いてありますが、
プロジェクトフォルダで下記コマンドを実行します。

    $npm install primeng --save

## PrimeNG を組み込む

PrimeNG のインストールはできたので昨日作ったアプリに PrimeNG を組み込んでみます。
まず、PrimeNG 用の css が用意されているのでそれを追加します。
css の定義を index.html に書いてみたのですが、
表示された画面で確認すると css ファイルが見つからない状態となります。
色々試行錯誤した結果、`src/style.css` ファイルに下記を追加することで期待した css が読み込まれることが分かりました。

```css
@import url('../node_modules/primeng/resources/themes/omega/theme.css');
@import url('../node_modules/primeng/resources/primeng.min.css');
```

読み込まれるというは正確ではなく、インポートした css が style.css ファイル内に展開されて、
それが html のヘッダに組み込まれるというのが正しい表現になります。
おそらくビルド時にそういうことをしているのだと思うのですが、
Angular のビルドフローがよくわかっていないのでなんとも言えません。
`src/style.css` に何も書いていないと html にも組み込まれないので何か判定があるのかと。

で、実はこれだけだと片手落ちで、[Font Awesome](http://fontawesome.io) で提供されている css と font 群を取り込まないと、
PrimeNG でのアイコン表示ができない状態となります。
まず、Font Awesome のサイトでフォントアイコンをダウンロードします。
プロジェクトの src フォルダ直下に resources フォルダを作り、
その下にダウンロードしたモジュール内にある、css フォルダと fonts フォルダをコピーします。
独自で取り込んだ css などをどこに配置するのが Angular として正しいのかよくわからないので、
PrimeNG の Showcace のソースコードと同じ構成にしてみました。

`src/style.css` にインポートを追加します。

```css
@import url('/resources/css/font-awesome.min.css');
```

## PrimeNG の Button コンポーネントを配置

事前の準備ができたので、PrimeNG のコンポーネントの中から Button コンポーネントを配置します。
`src/app/app.component.html` に下記のコードを追加します。

```html
<button pButton type="button" icon="fa-check" iconPos="left"></button>
```

次に `src/app/app.module.ts` を下記のように変更します。
diff がハイライトされていませんが差分表示です。

```
  import { BrowserModule } from '@angular/platform-browser';
  import { NgModule } from '@angular/core';
  import { FormsModule } from '@angular/forms';
  import { HttpModule } from '@angular/http';

  import { AppComponent } from './app.component';
+ import {ButtonModule} from 'primeng/primeng';

  @NgModule({
    declarations: [
      AppComponent
    ],
    imports: [
      BrowserModule,
      FormsModule,
-     HttpModule
+     HttpModule,
+     ButtonModule
    ],
    providers: [],
    bootstrap: [AppComponent]
  })
  export class AppModule { }
```

## 実行

実行すると下記の画面が表示されます。

![](/images/post_image_43.png)

## まとめ

PrimeNG のドキュメントを見ても module.ts ファイルを編集などは書いてないので、
この辺りのことは Angular を知っていればわかることなのかなぁとなんとなくモヤモヤした感じです。
実際仕組みがわかれば大したことをしていない気がするので、
もう少し色々試してみようと思います。
次はボタンへのイベントハンドリングをしてみようかと考えています。

## 参考

- [PrimeNG Demo](http://www.primefaces.org/primeng/#/button)

