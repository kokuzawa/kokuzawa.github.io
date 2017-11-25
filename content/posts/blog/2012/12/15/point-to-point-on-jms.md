---
title: "Point-to-Point on JMS"
date: 2012-12-15T15:50:00+09:00
tags: ["Java", "JavaEE", "GlassFish", "JMS"] 
---

[JavaEE Advent Calendar 2012](http://atnd.org/events/33783)の15日目のエントリーです。  
昨日は[@yoshioterada](https://twitter.com/yoshioterada)さんの[Java EE 7 WebSocket Client Sample Application with JavaFX](http://yoshio3.com/2012/12/14/javaee7-websocket-client-with-javafx/)です。  
明日は[@akirakoyasu](https://twitter.com/akirakoyasu)さんです。

## 普段は使わないJMSを使う

おそらくJMSの本来の利用方法は非同期通信を利用した分散処理なのだと思うけど、今回はそんな高尚な目的ではなく、単純なメッセンジャーとして利用します。
世の中のどの位のプロジェクトでJMSが利用されているのか分からないけど、JavaEEの仕様にJMSが含まれているにも関わらず、今まで本格的に利用した事がありません。
分散処理をするケースがあるプロジェクトに参加した事が無いのか、またはサーバがいつもTomcatだからなのか。
おそらく後者なのだと思うけど、ということはつまり分散処理の必要がないプロジェクトということなんだと思います。

そんな事もあって、JMSの知識が皆無だったわけですが、
JavaEEのアドベンドカレンダーをやるに当たって何か普段は触らないようなことをやりたいなと思い立ち、JMSを使ってみる事にしました。

## 必要なもの

* GlassFish-3.1.2.2
* jms-api-1.1-rev-1.jar

``` xml
<dependency>
  <groupId>javax.jms</groupId>
  <artifactId>jms-api</artifactId>
  <version>1.1-rev-1</version>
</dependency>
```

* imq-4.5.2.jar

``` xml
<dependency>
  <groupId>org.glassfish.mq</groupId>
  <artifactId>imq</artifactId>
  <version>4.5.2</version>
</dependency>
```

JMSを利用する為には、メッセージプロバイダが必要になります。プロバイダにはOpenMQ, MQSeries, SonicMQなどがありますが、
今回は導入が簡単なOpenMQを利用します。OpenMQはGlassFishに付属してインストールされます。インストール時に特に何かを意識する必要はありません。
また、OpenMQにアクセスする為に2つのライブラリが必要になります。jms-apiとimqです。Mavenリポジトリに登録されているので、
こちらも容易に入手可能です。

## 事前準備
GlassFishを起動しておく必要があります。ポートとして7676を利用するので、GlassFishがローカルではなく、リモート環境にある場合は、
ポートへのアクセスを許可する必要があるかもしれません。

## メッセージプロデューサーを作る
メッセージを送信するプロデューサーを作ります。

``` java
package jp.co.baykraft.jmsexample;

import com.sun.messaging.ConnectionConfiguration;
import com.sun.messaging.QueueConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Queue;
import javax.jms.QueueConnection;
import javax.jms.QueueSender;
import javax.jms.QueueSession;
import javax.jms.TextMessage;

/**
 * メッセージを送信するクラスです。
 * @author Katsumi
 */
public class Sender
{
    public static void main(String... args) throws JMSException
    {
        QueueConnectionFactory factory = new QueueConnectionFactory();
        factory.setProperty(ConnectionConfiguration.imqAddressList, 
            "localhost:7676");
        QueueConnection connection = factory.createQueueConnection();
        QueueSession session = connection.createQueueSession(false, 
            QueueSession.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue("KQueue");

        QueueSender sender = session.createSender(queue);
        TextMessage message = session.createTextMessage();
        message.setText("メッセージ");
        sender.send(message);

        connection.close();

        System.out.println("Finished.");
    }
}
```

`session.createQueue("KQueue")`でKQueueという文字列を指定していますが、これは任意の文字列です。
メッセージを受信する側でも同じ文字列を指定してメッセージを受け取ります。

## メッセージコンシューマを作る
次にメッセージを受信するコンシューマを作ります。

``` java
package jp.co.baykraft.jmsexample;

import com.sun.messaging.ConnectionConfiguration;
import com.sun.messaging.QueueConnectionFactory;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.Queue;
import javax.jms.QueueConnection;
import javax.jms.QueueReceiver;
import javax.jms.QueueSession;
import javax.jms.TextMessage;

/**
 * メッセージを受診するクラスです。
 * @author Katsumi
 */
public class Receiver
{
    public static void main(String... args) throws JMSException
    {
        QueueConnectionFactory factory = new QueueConnectionFactory();
        factory.setProperty(ConnectionConfiguration.imqAddressList, 
            "localhost:7676");
        QueueConnection connection = factory.createQueueConnection();
        QueueSession session = connection.createQueueSession(false, 
            QueueSession.CLIENT_ACKNOWLEDGE);
        Queue queue = session.createQueue("KQueue");

        QueueReceiver receiver = session.createReceiver(queue);
        receiver.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message msg)
            {
                try {
                    TextMessage message = (TextMessage) msg;
                    message.acknowledge();
                    System.out.println(message.getText());
                }
                catch (JMSException ex) {
                    Logger.getLogger(Receiver.class.getName())
                          .log(Level.SEVERE, null, ex);
                }
            }
        });

        connection.start();
    }
}
```

## 動かしてみよう
さて、コードも作ったので動かしてみます。
送信するメッセージはキューに溜め込まれるので、コンシューマを起動しておかなくても大丈夫です。
まずはプロデューサーを起動すると、「メッセージ」という文字列がメッセージプロバイダに送られます。

次にコンシューマを起動します。
コンシューマを起動すると、メッセージプロバイダのKQueueに溜め込まれたメッセージが受信されます。
コンシューマのコードの`QueueSession.CLIENT_ACKNOWLEDGE`の指定ですが、この指定の場合、
コンシューマ側で`TextMessage#acknowledge()`メソッドを呼び出すまで、キュー上のメッセージは削除されません。
コンシューマ側の処理が失敗した場合はキュー上のメッセージを削除したくないのでこの指定にしています。

## まとめ
GlassFishを起動するだけでメッセージプロバイダが起動されるのは非常に便利です。
メッセージプロバイダを起動しておけば、簡単なメッセージの送受信アプリが作れます。
キューの名前をコンシューマ毎に切り替えれば、複数クライアントのアプリも出来そうです。
JMSの本来の利用目的とは違うのかもしれませんが、こんな簡単なアプリから触ってみると楽しいと思います。
それにしても、GlassFishは最高ですね。

