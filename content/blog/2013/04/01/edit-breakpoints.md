---
title: "Edit breakpoint of the IntelliJ IDEA"
date: 2013-04-01T22:35:00+09:00
tags: ["IntelliJ IDEA"]
---
IntelliJも他のIDE同様デバッグが簡単にできます。  
もちろん複数のスレッドを同時にデバッグできるのですが、初期設定だと一つのスレッドが終わるまですべてのスレッドがSuspendされます。
最初はこういうものかなと思っていたんですが、設定で切り替えることができるようです。

## 手順
ブレークポイントをCtrl+クリックしてEditメニューを選択します。

![](/images/post_image_9.png)

表示されたポップアップのSuspendのところ、Allにチェックが入っていると思います。
これをThreadに変更します。もし今後すべての動作をThreadにするのであれば、右にある「Make Default」ボタンをクリックします。

![](/images/post_image_10.png)

これでスレッド毎にSuspendされるようになりました。

Best regards,  
Katsumi
