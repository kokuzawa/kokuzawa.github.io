---
title: "IntelliJ IDEAからDocker上のWildFlyでデバッグする"
date: 2016-02-28T01:56:31+09:00
tags: [Docker, WildFly, Java, "IntelliJ IDEA"] 
---
IntelliJ IDEAからDocker上のWildFlyコンテナにアプリケーションをデプロイし、
デバッグモードで起動することでステップ実行ができる環境を作ることが今回の目的です。
Docker上にコンテナを起動できる環境はできている前提になります。

<!-- MORE -->

## 環境

- OS: Mac OSX 10.11.3
- Java: Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
- メモリ: 16GB
- Docker version 1.9.1

## アプリの準備

どんなアプリでも良いのですが最終的な生成物はwarファイルにします。
今回はMavenプロジェクトで下記のようなフォルダ構成にします。

```
docker-wildfly-example/
├── Dockerfile
├── container_settings.json
├── docker-wildfly-example.iml
├── pom.xml
└── src
    └── main
        ├── java
        │   └── org
        │       └── katsumi
        │           └── HelloBean.java
        └── webapp
            ├── WEB-INF
            │   └── web.xml
            └── index.xhtml
```

肝心なDockerfileの内容は下記のようになります。

```
# WildFlyのイメージを取得
FROM jboss/wildfly:latest

# MAINTAINER
MAINTAINER Katsumi

# アプリケーションのデプロイ
COPY target/docker-wildfly-example.war /opt/jboss/wildfly/standalone/deployments/

# ポートの解放
EXPOSE 9999

# WildFlyの実行
CMD ["/opt/jboss/wildfly/bin/standalone.sh", "-b", "0.0.0.0", "--debug", "9999"]
```

`mvn package`することで生成される`target/docker-wildfly-example.war`ファイルを
コンテナ上の`/opt/jboss/wildfly/standalone/deployments/`に配置することで
WildFly起動時に自動的にデプロイさせます。
また、デバッグ用のポートとして`9999`を使用するため`EXPOSE`に指定します。

デバッグはリモートデバッグをすることになるので、
WildFly起動オプションに`--debug`を付与して`9999`ポートを指定します。

## 実行環境の構築

IntelliJ IDEAにDocker PluginがインストールされているとRun/Debug ConfigurationsにDocker Deploymentを追加することができます。

![](/images/post_image_41.png)

Debugポートに9999を指定すると画面下部にワーニングが表示されます。

    Warning: Debug port forwarding not found

このワーニングの右側にFixボタンを表示されるのでこれをクリックすると
port設定をしたjsonファイルの保存先を聞かれるので任意の場所に保存します。
アプリケーションツリーにあるcontainer_settings.jsonがそれになります。
このファイルはContainerタブのJSON fileの項目に設定されます。

やっていることはContainerタブのport bindingsで9999ポートを追加したのと同じことなのですが、
port bindingsに設定してもワーニングが消えません。
ワーニングは消えなくてもport bindingsの設定は有効になるので
ワーニングが気にならないようであればport bindingsに設定しても良いです。
ただし注意点としてjsonファイルとport bindingsの両方を指定するとport bindingsの方の設定が無視されるようです。
9999以外のポートをバインドする場合は注意する必要があります。

これでデバッグ起動すればステップ実行ができるようになります。

