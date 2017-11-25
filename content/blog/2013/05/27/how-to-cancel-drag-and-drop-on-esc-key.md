---
title: "how to cancel drag-and-drop on esc key"
date: 2013-05-27T01:12:00+09:00
tags: [Flex]
---
Flex-4.5.1で、ツリーノードのDrag and DropをEscキーでキャンセルしてみよう、という話です。  
標準機能としてはありませんが、これ以降のバージョンで対応されている可能性もあるという前提になります。

実はDrag and DropをEscキーを使ってキャンセルできることは知りませんでした。
Windowsでもできるようですが、普段使っているMacでもできることにさっき気がついたぐらいなので、
そんな操作を利用している人は少ないのではないかと思ってました。
ですが、検索してみると意外と多くの人が利用しているようです。

## 実装する
機能が無いので実装することにします。

``` xml
<?xml version="1.0"?>
<s:Application xmlns:fx="http://ns.adobe.com/mxml/2009"
               xmlns:s="library://ns.adobe.com/flex/spark"
               xmlns:mx="library://ns.adobe.com/flex/mx"
               creationComplete="creationComplete(event)">

    <fx:Script><![CDATA[
        import mx.events.FlexEvent;
        import mx.managers.DragManager;
        import mx.managers.dragClasses.DragProxy;

        protected function creationComplete(event:FlexEvent):void
        {
            addEventListener(KeyboardEvent.KEY_DOWN, function (event:KeyboardEvent):void
            {
                if (event.keyCode == Keyboard.ESCAPE) {
                    DragManager.acceptDragDrop(null);
                    myTree.hideDropFeedback(null);
                    myTree.dispatchEvent(new MouseEvent(MouseEvent.MOUSE_UP));
                    var dragProxy:DragProxy = DragManager.mx_internal::dragProxy;
                    if (dragProxy) {
                        dragProxy.mouseUpHandler(new MouseEvent(MouseEvent.MOUSE_UP));
                    }
                }
            });
        }
        ]]></fx:Script>

    <s:layout>
        <s:HorizontalLayout/>
    </s:layout>

    <mx:Tree id="myTree"
             width="200"
             height="300"
             dragMoveEnabled="true"
             dragEnabled="true"
             dropEnabled="true"
             labelField="@label">
        <mx:XMLListCollection id="treeData">
            <fx:XMLList>
                <node label="Mail Box">
                    <node label="Inbox">
                        <node label="AAAAA"/>
                        <node label="BBBBB"/>
                        <node label="CCCCC"/>
                    </node>
                    <node label="Outbox">
                        <node label="DDDDD"/>
                        <node label="EEEEE"/>
                    </node>
                </node>
            </fx:XMLList>
        </mx:XMLListCollection>
    </mx:Tree>

</s:Application>
```

大したことはしてなくて、重要なのは下記の部分。

``` javascript
addEventListener(KeyboardEvent.KEY_DOWN, function (event:KeyboardEvent):void
{
    if (event.keyCode == Keyboard.ESCAPE) {
        DragManager.acceptDragDrop(null);
        myTree.hideDropFeedback(null);
        myTree.dispatchEvent(new MouseEvent(MouseEvent.MOUSE_UP));
        var dragProxy:DragProxy = DragManager.mx_internal::dragProxy;
        if (dragProxy) {
            dragProxy.mouseUpHandler(new MouseEvent(MouseEvent.MOUSE_UP));
        }
    }
});
```

簡単に説明すると、まず、ツリーコンポーネントの上以外でもEscキーをハンドリングする必要があるので、キーイベントのリスナーはApplicationに付けます。
で、Escが叩かれたらdrag-and-dropをキャンセルさせるためにコンポーネントに通知します。

`DragManager.acceptDragDrop(null);`でドロップを無効に、ドラッグすると移動先にラインが表示されるので、
`myTree.hideDropFeedback(null);`でラインを消します。
ドラッグ中はマウスをダウンしたままなので、キャンセルに合わせてツリーコンポーネントに対して
`myTree.dispatchEvent(new MouseEvent(MouseEvent.MOUSE_UP));`を通知します。

そのあとがちょっと特殊で、mx_internalでDragManagerの内部変数を参照し、マウスアップイベントを通知します。
この通知が無いと、ドラッグ操作をキャンセルしたと認識されません。

もっと良いやり方や、最新バージョンだと出来るよとか教えて頂けると嬉しいです :D

