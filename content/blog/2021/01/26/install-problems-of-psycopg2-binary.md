---
title: "[Homebrew] psycopg2-binaryインストールの問題"
date: 2021-01-26T23:20:29+09:00
tags: [Homebrew, Python]
draft: false
---

## 環境

* OS: MacOS Big Sur
* チップ: Apple M1
* メモリ: 16GB
* Python-3.8.2

## やりたいこと

homebrewでpsycopg2-binaryをインストールしたい。

## やったこと

以下のコマンドを実行。

```
brew install psycopg2-binary
```

以下のエラー（抜粋）が発生。

```
3.8/psycopg/typecast.o -L/opt/homebrew/lib -lpq -lssl -lcrypto -o build/lib.macosx-10.14.6-arm64-3.8/psycopg2/_psycopg.cpython-38-darwin.so
    ld: library not found for -lssl
    clang: error: linker command failed with exit code 1 (use -v to see invocation)
    error: command 'clang' failed with exit status 1
    ----------------------------------------
```

## 解決方法

.zshrcに以下を追加。

```
export LDFLAGS="-L/opt/homebrew/opt/openssl/lib"
export CPPFLAGS="-I/opt/homebrew/opt/openssl/include"
```

ただし、opensslがインストールされている前提。インストールされていない場合は下記コマンドで事前にopensslをインストールしておく。

```
brew install openssl
```

## 参考

* [error installing psycopg2, library not found for -lssl](https://stackoverflow.com/questions/26288042/error-installing-psycopg2-library-not-found-for-lssl)

