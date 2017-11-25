---
title: "はじめてのLATEX"
date: 2012-11-24T01:25:00+09:00
tags: [Mac, LaTeX] 
---

LaTeXって良く目にするんだけれども、それが何なのか、文書作成用の何か程度の知識しかなかった。
何がきっかけだったのかすっかり忘れてしまったけど、今日LaTeXを使ってみようと思い立ち、利用するための環境設定を色々と調べたので、その記録を書き記しておこうと思う。

## LaTeXって何？
> LATEX（ラテック、ラテフ、レイテック、レイテックス）とは、レスリー・ランポート (英: Leslie Lamport) によって開発されたテキストベースの組版処理システムである。電子組版ソフトウェア TEX にマクロパッケージを組み込むことによって構築されており、単体の TEX に比べて、より手軽に組版を行うことができるようになっている。なお、LATEX を基にアスキーが日本語処理に対応させたものとして日本語 LATEX が、さらに縦組み処理にも対応させたものとして pLATEX がある。

[Wikipedia](http://ja.wikipedia.org/wiki/LaTeX)によれば、組版処理システムということらしい。Wordとかの所謂文書スタイルを制御するものと考えれば良さそう。
ただ、これだけではLaTeXが一体なんなのかがぼんやりとしか分からない。
やっぱり実際に触ってみた方が良さそうだ。

## MacにLaTeXをインストールする
MacでLaTexを使うには何をインストールしたら良いのか？
調べたところ、pTeXというのを入れれば良く、これは縦組み処理も対応したpLATEXと同じもの。
早速インストールしてみよう。インストーラを見つけるよりは、MacPortsを利用した方が早いんじゃないのかと思い、MacPortsで検索してみる。

    > sudo port search ptex
    
    pTeX @20110314 (tex, print, textproc, japanese)
        Japanese TeX (pTeX) processing environment
    
    ptex-sfmacros @0 (tex, print, japanese)
        Tategumi/Tateyoko/Kunten packages written by Shinsaku Fujita.

2つ見つけたけど、1つ目の方だろう。インストールする場合はオプションをつけた方が良いとのこと。

    > sodo port install pTeX +utf8 +motif

長い...。とても時間がかかるので最初にコーヒーを淹れておけばよかった。ゆっくり飲みながら気長に待つ。
インストールが終わったら、ちゃんとインストールがされたか確認する。

    > platex -version
    
    TeX 3.141592-p3.1.10 (utf8.euc) (Web2C 7.5.4)
    kpathsea version 3.5.6
    ptexenc version 0.997
    Copyright (C) 1997-2004 D.E. Knuth.
    Kpathsea is copyright (C) 1997-2004 Free Software Foundation, Inc.
    There is NO warranty.  Redistribution of this software is
    covered by the terms of both the pTeX copyright and
    the GNU General Public License.
    For more information about these matters, see the files
    named COPYING and the pTeX source.
    Primary author of TeX: D.E. Knuth.
    Kpathsea written by Karl Berry and others.

ああ、やっとインストールできた。

## 使ってみよう
LaTeXを使うには、まず素となるテキストを作る必要がある。

    \documentclass[a4paper,12pt]{jarticle}
    \begin{document}
    本文です。
    \end{document}

firstdoc.texという名前で保存。tex拡張子はおそらく慣習？
これをplatexでコンパイルする。コンパイルという言葉が正しいかどうかは分からないけど。

    > platex firstdoc.tex

すると、拡張子がDVIの中間ファイルが生成される。
このファイルはプレビューすることができて、どんな組版になったのかを確認できるようだ。

    > xdvi firstdoc.dvi
    
    Error: Can't open display:

エラーが出た。Macの場合、ターミナルでプレビューを見ようとしてもダメらしく、
プレビューを見るにはX11をインストールする必要があるらしい。
今更X11はインストールしたくないので、別の方法で確認してみようと思う。

    > dvipdf firstdoc.dvi

これは、PDFファイルを生成するためのコマンド。実行するとPDFファイルが生成されるので、それを開く。

    > open firstdoc.pdf

これでプレビューアプリでfirstdoc.pdfを開くことができる。
PDFは開いたままファイルを修正して、platex -> dvipdf でもう一回PDFを生成すると、
開いているPDFが更新される。これで組版の確認をすることができるようになった。

## まとめ
たいした内容を書かなくても、それっぽいドキュメントを作ることができるのはありがたい。
あと、文章を作る行程がプログラミングのようで、これもなかなか楽しいし、
素となるファイルがバイナリではないので、版数管理もし易そうだ。
組版をするにはいろいろと覚えることが多そうだけど、それを差し引いてもすばらしい。

## 参考サイト
<http://keizai.xrea.jp/latex/tutorial/index.html>

