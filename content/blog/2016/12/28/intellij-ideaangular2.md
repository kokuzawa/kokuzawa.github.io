---
title: "IntelliJ IDEAでAngular2アプリを動かす"
date: 2016-12-28T00:39:11+09:00
tags: [IntelliJ IDEA, Angular2]
---
Angular2のアプリをIntelliJ IDEAで動かすまでの試行錯誤の記録です。

<!-- more -->

## 環境

- macOS Sierra
- IntelliJ IDEA 2016.3

## Angular2のインストール

Angular2を動かすのにはNode.jsが必要なので、最初にNode.jsをインストールします。
macOSの場合、homebrewでnodebrewを入れるのが良さそうです。
nodebrewはNode.jsのバージョン管理システムだそうです。

    $brew install nodebrew

nodebrewインストール直後だと、カレントのNode.jsが選択されていないため、
Node.jsのコマンドを打ってもそんなコマンドはないと怒られてしまいます。
そこで、カレントのNode.jsを選択します。
選択方法ですが、まずインストールされているNode.jsのバージョンを知る必要があるので、
下記コマンドを実行し、バージョンリストを表示します。

    $nodebrew list

バージョンが表示されたら、利用したいバージョンを指定した下記コマンドを実行します。
私の環境ではv7.3.0がインストールされていたので、これをカレントにします。

    $nodebrew use v7.3.0

これでNode.jsが使えるようになりました。

次に目的のAngular2をインストールするわけですが、
ここではコマンドラインツールであるangular-cliをインストールします。
インストールはnpm(Node Package Manager)を使います。

    $npm install -g angular-cli

これでAngular2アプリを作る準備ができました。

## Angular2アプリを作る

早速アプリを作りたいのですが、IntelliJ IDEAではAngularプロジェクトを作るメニューはありません。
プラグイン等であるのかもしれませんが、プレーンな状態ではできないので、
Angular-cliを使って雛形のプロジェクトを作成します。
プロジェクトを作りたいフォルダに移動して下記コマンドを実行します。

    $ng new angular-start

これで雛形のアプリが作成されました。
angular-startフォルダに移動して下記コマンドを実行すると、`http://localhost:4200/`で最初の画面が表示されます。

    $ng serve

## IntelliJ IDEAに読み込む

ここまでの作業でAngular2の動作するアプリはできましたが、
IntelliJ IDEAでの編集はまだできません。
IntelliJ IDEAで編集するために下記手順でプロジェクトを読み込みます。

1. File -> New -> Project from Existing Sources... を選択してangular-startフォルダを選択する
2. Import ProjectダイアログでCreate Project from existing sourcesを選択してNextボタンをクリックする
3. 以降は何も変更せず、Nextボタンをクリックする
4. Finishボタンが表示されたらそれをクリックする

これでソースの読み込みは完了しました。

## IntelliJ IDEAで実行する

Run -> Edit Configurations... を選択してRun/Debug Configurationsダイアログを表示します。
下記イメージではnpmの実行設定を既に作成した状態になります。

![](/images/post_image_42.png)

イメージ右側のNode Interpreterの部分、選択された状態となっていますが、
これはIntelliJ IDEAにNodeJSプラグインをインストールしていないと選択されません。
ですので、事前にNodeJSプラグインをインストールする必要があります。

package.json, Scripts, Argumentsの部分が空白となっていると思うので、
イメージ内で設定しているように設定します。
これで実行する準備が整いました。
実行するとコマンドラインで`ng serve`とした時と同様のログがコンソールに出力されるのが確認できると思います。
最初の時と同じように、起動後`http://localhost:4200`にアクセスすることで最初の画面を表示することができます。

## まとめ

コマンドラインからプロジェクトを作った後にIntelliJ IDEAに読み込ませるのがちょっと面倒です。
プラグインでAngular2のプロジェクトが作れるようになると良いのですが、
今の所できなさそうです。（どこかにあるのかな？）
ただし、一度読み込ませてしまえば、後は使い慣れたIntelliJ IDEA上で編集ができるし、
補完も効くので開発効率が良さそうです。

## 参考

- [Start Angular](https://www.gitbook.com/book/albatrosary/start-angular/details)


