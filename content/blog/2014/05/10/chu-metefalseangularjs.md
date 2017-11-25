---
title: "初めてのAnglarJS"
date: 2014-05-10T18:50:20+09:00
tags: [AngularJS] 
---
AngularJS + JAX-RS + JPA でアプリを作ってます。
今回 AngularJS を初めて使っているんですがとても便利ですね。
どの辺りがと言われると、具体的に答えられるほど使い込んでいないので答えに窮するのですが、
今分かっている範囲では、テストが簡単にできるのが良い感じです。

<!-- MORE -->

## 基本
AngularJS を利用するためには当たり前ですが、jsファイルを呼び出す必要があります。
これは AngularJS の[公式サイト](https://angularjs.org)からダウンロードできるので、そこから取得しました。
ダウンロードしたファイルを任意の場所に配置し、下記を `head` タグの間に追加します。

``` html
<script src="angular.js"></script>
```

あと、下記の `ng-app` を忘れずに `html` タグに追加します。
これは、このページが AngularJS のページだよと宣言するためのディレクティブだそうです。

``` html
<html ng-app>
```

## 初期値の設定
AngularJS は `ng-repeat` というディレクティブを利用することで繰り返し処理をすることができます。
繰り返し処理で出力するデータは下記のように `ng-init` を利用して初期値を設定することができます。
まあ、下記のコードだと繰り返し処理をさせる意味はあまりないのですが...。

``` html
<html ng-app>
  <head>
    <script src="angular.js"></script>
  </head>
  <body>
    <div ng-init="names = ['Taro', 'Jiro', 'Hanako']">
      <div ng-repeat="name in names">
        {{name}}
      </div>
    </div>
  </body>
</html>
```

これを実行すると以下のような画面が表示されます。

    Taro
    Jiro
    Hanako

とても簡単です。
ここで表示している Taro, Jiro, Hanako の値を API から取得して表示しようとした場合は下記のようなコードになります。

{% raw %}
``` html
<html ng-app="myApp">
  <head>
    <script src="angular.js"></script>
    <script src="angular-resource.js"></script>
    <script>
      var myApp = angular.module('myApp', ['ngResource']);

      myApp.factory('testService', ['$resource', function($resource)
      {
        return $resource('/test/rest/names', {query: {method: 'GET', isArray: true}});
      }]);

      myApp.controller('myCtrl', ['testService', '$scope', function(testService, $scope)
      {
        $scope.names = testService.query();
      }]);
    </script>
  </head>
  <body>
    <div ng-controller="myCtrl">
      <div ng-repeat="name in names">
        {{name}}
      </div>
    </div>
  </body>
</html>
```
{% endraw %}

### 参考
以下のサイトを参考にしました。  

- [AngularJS 公式サイト](https://angularjs.org)
- [ドットインストール - AngularJS入門](http://dotinstall.com/lessons/basic_angularjs)

