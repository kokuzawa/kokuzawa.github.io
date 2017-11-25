---
title: "Dockerで始めるVMを利用した開発"
date: 2016-02-11T16:49:26+09:00
tags: [Docker] 
---
結構前からDockerの事を聞いていてそれは仮想化技術だと認識していました。
なんで今まで触ってこなかったのかというと仮想化環境を作るにはそれなりの
マシンスペックが必要なのだろうと。つまり貧弱なマシンを使っている僕には関係ない。
自宅のMac Book Proが壊れて新しくなったり、
会社のPCのスペックが上がったりしたのでこれは触りどきかと思って今手をつけてみたわけです。
というわけでまさに触り始めなわけで開発まではたどり着いていません。タイトル嘘という方向で。

<!-- MORE -->

最初に何をしたかというと、DockerToolBoxというやつをインストールしました。
Homebrewでも入れられるみたいですがどこかのサイトにHomebrewで入れるとなかなか最新にならないよという
至極まっとうな記載があったのでひとまず最新がいいなあと思い、インストールモジュールをダウンロードしてみました。

これ、インストールすると目に見えて分かるのは３つのソフトがインストールされるということです。

- Docker Quickstart Terminal
- Kitematic (Beta)
- VirtualBox

もしかしたら見えないところに他のソフトがインストールされているのかもしれませんが、
まだよくわかっていません。

最初にDocker Quickstart Terminalを起動します。
これはOSXのターミナルが起動します。ここの中でCUIで操作するようです。
で、ちょっとハマったのは起動したターミナルに別タブを開いてそこでDockerの起動とかしようとしても
仮想マシンに接続できないようで理解するまで時間がかかりました。

Docker Quickstart Terminalを起動すると仮想マシンがdefaultという名前で起動します。
これはVirtualBoxを起動するとわかります。
今のところVirtualBoxを使って何かするということはなさそうだという理解です。
このdefaultの仮想マシンですが、間違ってログアウトしてしまったらログインのIDとパスワードがわからなくて難儀しました。
どうやらCore Linuxというものを使っているらしくそれのデフォルトのIDとパスワードでログインできるようです。
こういうDockerとは直接関係ない機能を試してみたくなるところが僕の悪いところで、
Dockerそのものをまだちゃんと触れていない状態です。

Kitematicはいろいろな人がアップしたDockerイメージが登録されているDockerHubというところへの
アクセスをGUI経由でできるソフトのようです。
Dockerイメージとかもうよくわからないので、この本を買ってちゃんと勉強することにしました。

<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=FFFFFF&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=moonwhaleblog-22&o=9&p=8&l=as1&m=amazon&f=ifr&ref=qf_sp_asin_til&asins=479814102X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
<br/>
<br/>

まだ最初の方しか読んでいませんが、
ざっくり言うとMavenみたいな感じですね！多分。

というわけでこれからしばらくはDockerを使ってみたいと思っています。


