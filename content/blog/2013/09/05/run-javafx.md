---
title: "バブルソートをJavaFXで可視化"
date: 2013-09-05T03:15:00+09:00
tags: [JavaFX] 
---
『アルゴリズムとデータ構造』という本の一番最初に出てくるバブルソートを可視化したいと思い、JavaFXでアプリケーションを作った。
作るのはそれほど難しい話ではないので、ちょっとした解説を加えつつ紹介しようと思う。
 
実行モジュールもあとで提示するので確認して頂ければ幸い。

<!-- MORE -->

IDEはIntelliJ IDEAを使った。IntelliJ IDEAにはJavaFXモジュール用のプロジェクトを作ることができるのでそれを利用する。
ビルドすると親切なことにjarファイルだけでなく、JNLPやHTMLファイルを生成してくれるので非常に便利。

ビルドされた実行モジュールを見てもらった方が分かり易いと思い、
Appletとしてブラウザ上で動かそうと思ったのだが、
IDEAが生成してくれたHTMLをSafariで開いても、いつまでたっても起動しない。
HTMLのソースを見てみると、dtjava.jsというJavaScriptを使って起動しようとしているようだが、
少なくともSafariではうんともすんとも言わない。
他のブラウザなら動作するのだろうかと思い、Chromeで実行すると、今度は「このブラウザではサポートされていない」という趣旨の
メッセージが表示される。多分Macだとダメなのだろう。もし起動する方法を知っていたら教えて頂きたい。

ということで、Appletとして動かすことは諦めたので、JNLPファイルへのリンクを提示する。

[JavaFXApp.jnlp](https://dl.dropboxusercontent.com/u/46295066/javafx/BubbleSort/JavaFXApp.jnlp)

このJNLPをダウンロードして起動すると、バブルソートが実行される。
OSXの場合だと、セキュリティによって起動できないかもしれない。
その場合、ダウンロードしたJNLPファイルをCtrl＋クリックして「このアプリケーションで開く」-> 「Java Web Start」とメニューを
選択して行けば、起動しても良いか尋ねてくれるので、ここから起動が出来ると思う。

## アプリについて

![](/images/post_image_15.png)

ソート対象となる100本の黒ラインはランダムに並んでいて、現在処理中のラインを赤で表している。
実際に動作を見ると、短い黒ラインがだんだんと左側に寄って行くのが分かる。

## コード
初期データは`Collections.shuffle(Collection)`でシャッフルする。

``` java
for (int i = 1; i <= N; i++) {
    data.add(i);
}
Collections.shuffle(data);
```

ステージの初期化。黒ラインと赤ラインを表すPathを2つ。
repaint()メソッドで描画した結果から必要になった幅をステージの幅として設定する。+10はパディング分になる。

``` java
path.setStrokeWidth(3.0);
path.setStroke(Color.GRAY);
currentPath.setStrokeWidth(4.0);
currentPath.setStroke(Color.RED);
root.getChildren().add(path);
root.getChildren().add(currentPath);

final int width = repaint();

primaryStage.setTitle("Bubble Sort");
primaryStage.setResizable(false);
primaryStage.setScene(new Scene(root, width + 10, VIEW_HEIGHT));
primaryStage.show();
```

実際のライン描画処理。
JavaFXではTimelineクラスを使うと秒毎の処理が書けるので簡単にアニメーションを作ることができる。
待機時間無しだとソートが一瞬で終わってしまい、過程を見ることができないので、
ここでは待機時間に10ミリ秒を指定している。`new Duration(10)`の部分。

``` java
timeline.getKeyFrames().addAll(
        new KeyFrame(new Duration(10), new EventHandler<ActionEvent>()
        {
            @Override
            public void handle(ActionEvent actionEvent)
            {
                if (data.get(counter) > data.get(counter + 1)) {
                    flag = true;
                    Collections.swap(data, counter, counter + 1);
                }

                counter++;

                repaint();

                if (counter >= N - 1 - k) {
                    if (!flag) {
                        repaint();
                        timeline.stop();
                    }
                    flag = false;
                    counter = 0;
                    k++;
                }
            }
         })
);
timeline.setCycleCount(Timeline.INDEFINITE);
timeline.play();
```

バブルソートなので入れ替えが発生しなくなるまでソートを繰り返す。
入れ替えの発生有無は`flag`によって制御している。この`flag`がfalseのまま最終ラインまで処理されると、
`timeline.stop()`を呼び出し、タイムラインを停止する。

入れ替えが発生した場合は、下記のrepaint()メソッドを呼び出し、
画面の再描画を行っている。
JavaFXでラインを引く場合、`MoveTo`クラスで座標を移動し、`LineTo`クラスで指定座標までのラインを引く。

``` java
int repaint()
{
    path.getElements().clear();
    currentPath.getElements().clear();
    int position = 10;
    int localCounter = 0;
    for (Integer i : data) {
        if (counter == localCounter) {
            currentPath.getElements().add(new MoveTo(position, (VIEW_HEIGHT - 10)));
            currentPath.getElements().add(new LineTo(position, ((VIEW_HEIGHT - 10) - (i * 2))));
        }
        else {
            path.getElements().add(new MoveTo(position, (VIEW_HEIGHT - 10)));
            path.getElements().add(new LineTo(position, ((VIEW_HEIGHT - 10) - (i * 2))));
        }
        position += 5;
        localCounter++;
    }

    return position;
}
```

ソースコードはbitbucketにあるので完全版はそちらを参照してください。
[https://bitbucket.org/kokuzawa/bubblesort](https://bitbucket.org/kokuzawa/bubblesort)

