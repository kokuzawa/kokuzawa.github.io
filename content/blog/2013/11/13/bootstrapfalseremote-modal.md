---
title: "BootstrapのRemote Modal"
date: 2013-11-13T03:55:00+09:00
tags: [bootstrap] 
---
bootstrap-3.0.2のRemote Modalの書き方がやっと分かった。  
今までは.modal-bodyだけが書き変わっていたのに、バージョン3以降は中身が全部書き変わるみたい。  

つまり呼び出し元はこれだけでいい。

``` html
<a href="inner.html" data-toggle="modal" data-target="myModal">Click</a>
<div class="modal fade" id="myModal"></div>
```

呼び出し先に実際の内容を書く。

**inner.html:**

``` html
<div class="modal-dialog">
  <div class="modal-content">
    <div class="modal-header">
      <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&#215;</button>
      <h4 class="modal-title" id="editModalLabel">${msg['mypage.edit_time']}</h4>
    </div>
    <div class="modal-body">
      <p>Loading...</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
    </div>
  </div>
</div>
```

分かってみれば簡単だったけど、ドキュメントからは読み取れなかったなぁ。

### Add Nov 27, 2013
このままだとinner.htmlの内容が動的に変わる場合、最初にモーダルを表示した内容から書き変わらないので、
以下のスクリプトを追加する必要があります。

``` javascript
$('#myModal').on('hidden.bs.modal', function() {
    $(this).removeData('bs.modal');
});
```

Best Regards.

