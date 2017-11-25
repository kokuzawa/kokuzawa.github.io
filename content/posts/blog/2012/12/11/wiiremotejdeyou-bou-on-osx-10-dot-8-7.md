---
title: "WiiRemoteJで遊ぼう on OSX 10.8.7"
date: 2012-12-11T01:46:00+09:00
tags: [Java, Mac, Wii] 
---

[Java Advent Calendar 2012](http://atnd.org/events/33871)の11日目のエントリーです。  
昨日は[@cero_t](https://twitter.com/cero_t)さんです。  
明日は[@snuffkin](https://twitter.com/snuffkin)さんです。

## クリスマスだから楽しいことをしよう
ということで、WiiUも発売したことだし、今更ながら、WiiRemoteJを取り上げてみたいと思います。
WiiRemoteJはBluetoothを利用して、WiiリモコンでJavaのアプリを操作するためのライブラリです。
最新版はv1.6というのがあるようなのですが、見つけることができなかったため、v1.4を使ってみたいと思います。

## OSX 10.8.7 Mountain Lionで動かす
今回やりたいことは、Macbook Pro上にWiiリモコンのレシーバーとなるアプリを起動し、
Wiiリモコンを使ってそのアプリを操作する、ということです。
レシーバーアプリは下記2つのライブラリが必要になります。

* [BlueCove.jar](https://code.google.com/p/bluecove/)
* [WiiRemoteJ.jar](http://dl.qj.net/wii/homebrew/wiiremotej-v14.html)

BlueCoveはJavaのBluetoothを利用する為のAPIの規約であるJSR-82の実装ライブラリです。
WiiRemoteJを動かす為に必要になります。
BlueCoveの最新版は2.1.0です。ところがこのBlueCove-2.1.0、Mountain Lion上では動かす事ができません。
Mountain Lionで動かす為には、まだ正式にリリースされていないBlueCove-2.1.1-SNAPSHOTを利用する必要があります。

ただ、この2.1.1-SNAPSHOTも問題があります。
BlueCoveはBluetoothにアクセスする為に/System/Library/Frameworks/IOBluetooth.frameworkというフレームワークを利用しているのですが、
BlueCove-2.1.1-SNAPSHOTはこのIOBluetooth.frameworkに対応できていないため、実行時にエラーが発生してしまいます。

という訳で、このままでは実行できそうにありません。
そこで、[ここ](https://groups.google.com/forum/#!msg/bluecove-users/7jWv1V1GC-4/jCHnASj1pbMJ)で提供されている、
BlueCove-2.1.1-SNAPSHOTで操作できるIOBluetooth.frameworkに置き換えることにします。
置き換える事により、他のアプリで問題が発生するかもしれません。
置き換える前に、元のIOBluetooth.frameworkのバックアップを作る事をお勧めします。

さあ、ここまでできれば、あとはレシーバーを作るだけです。

## レシーバーを実装する
下記が実装コードになります。動作は単純で、1ボタン、2ボタン、マイナスボタン、プラスボタン、Aボタン、Bボタン、十字キーを押した場合は、
それを標準出力に表示、ホームボタンを押したらレシーバーを終了します。

``` java
import java.io.IOException;
import java.util.logging.Level;
import java.util.logging.Logger;
import wiiremotej.WiiRemote;
import wiiremotej.WiiRemoteJ;
import wiiremotej.event.WRAccelerationEvent;
import wiiremotej.event.WRButtonEvent;
import wiiremotej.event.WRStatusEvent;
import wiiremotej.event.WiiRemoteAdapter;
import wiiremotej.event.WiiRemoteDiscoveredEvent;
import wiiremotej.event.WiiRemoteDiscoveryListener;

/**
 * WiiRemoteJサンプルアプリ
 * @author Katsumi
 */
public class Wii extends WiiRemoteAdapter implements WiiRemoteDiscoveryListener
{
    private WiiRemote _remote;

    public static void main(String... args)
    {
        Wii wii = new Wii();
        WiiRemoteJ.findRemotes(wii, 1);
    }

    @Override
    public void wiiRemoteDiscovered(WiiRemoteDiscoveredEvent evt)
    {
        _remote = evt.getWiiRemote();
        try {
            _remote.setAccelerometerEnabled(true);
            _remote.setSpeakerEnabled(true);
            _remote.setLEDIlluminated(0, true);
        }
        catch (IOException | IllegalStateException ex) {
            Logger.getLogger(Wii.class.getName()).log(Level.SEVERE, null, ex);
            if (null != _remote && _remote.isConnected()) {
                _remote.disconnect();
            }
        }

        _remote.addWiiRemoteListener(this);
    }

    @Override
    public void disconnected()
    {
    }

    @Override
    public void findFinished(int numFound)
    {
    }

    @Override
    public void statusReported(WRStatusEvent evt)
    {
    }

    @Override
    public void accelerationInputReceived(WRAccelerationEvent evt)
    {
    }

    @Override
    public void buttonInputReceived(WRButtonEvent evt)
    {
        if (evt.wasPressed(WRButtonEvent.TWO)) {
            System.out.println("2");
        }
        else if (evt.wasPressed(WRButtonEvent.ONE)) {
            System.out.println("1");
        }
        else if (evt.wasPressed(WRButtonEvent.B)) {
            System.out.println("B");
        }
        else if (evt.wasPressed(WRButtonEvent.A)) {
            System.out.println("A");
        }
        else if (evt.wasPressed(WRButtonEvent.MINUS)) {
            System.out.println("Minus");
        }
        else if (evt.wasPressed(WRButtonEvent.PLUS)) {
            System.out.println("Plus");
        }
        else if (evt.wasPressed(WRButtonEvent.LEFT)) {
            System.out.println("Left");
        }
        else if (evt.wasPressed(WRButtonEvent.RIGHT)) {
            System.out.println("Right");
        }
        else if (evt.wasPressed(WRButtonEvent.DOWN)) {
            System.out.println("Down");
        }
        else if (evt.wasPressed(WRButtonEvent.UP)) {
            System.out.println("Up");
        }
        else if (evt.wasPressed(WRButtonEvent.HOME)) {
            if (null != _remote && _remote.isConnected()) {
                _remote.disconnect();
            }
            System.exit(0);
        }
    }
}
```

## 動かしてみる
レシーバー起動時のVMオプションに下記を設定します。
このオプションを設定しないと、実行時にエラーが発生します。

```
-Dbluecove.jsr82.psm_minimum_off=true
```

レシーバー起動後にWiiリモコンの電池脇にある赤いSyncボタンを押す必要があります。数秒待つとアプリとWiiリモコンが繋がります。

実際に動かしているときの動画です。

<iframe width="560" height="315" src="https://www.youtube.com/embed/DzjeZHDwMFI" frameborder="0" allowfullscreen></iframe>


## まとめ

本当はボタンを押したらWiiリモコンで音を鳴らすということもしてみたかったのですが、
エラーが出て音源ファイルがWiiリモコンにうまく転送されませんでした。
仕組みとしては、アプリ側からのアクションでWiiリモコン側で音を鳴らす事もできるはずです。
音が出せるようになれば、ちょっとしたパーティーゲームができそうなので、この時期にはもってこいの遊びではないでしょうか。

