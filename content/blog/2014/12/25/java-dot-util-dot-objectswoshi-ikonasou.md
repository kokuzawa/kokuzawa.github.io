---
title: "java.util.Objectsを使おう"
date: 2014-12-25T01:30:39+09:00
tags: [Java]
---

この記事は[Java Advent Calendar 2014](http://qiita.com/advent-calendar/2014/java)の25日目です。  
昨日は[@smogami](https://twitter.com/smogami/)さんの「[LombokとLombok-pg： Javaコードを減量する魔法のスパイス](http://blog.exoego.net/2014/12/lombok-and-lombok-pg-to-reduce-fat-java.html)」でした。

今年も様々な面白い記事がアップされました。
そんな楽しかった25日間も今日で最終日です。
最終日なのですが、ちょっと地味にJava7で追加されたjava.util.Objectsについて書きたいと思います。

<!-- MORE -->

java.util.Objectsクラスでは今までありそうでなかった便利なユーティリティメソッドが用意されています。
Java7の時点では下記のメソッドが定義されています。

- static <T> int compare(T a, T b, Comparator<? super T> c)
- static boolean deepEquals(Object a, Object b)
- static boolean equals(Object a, Object b)
- static int     hash(Object... values)
- static int     hashCode(Object o)
- static <T> T   requireNonNull(T obj)
- static <T> T   requireNonNull(T obj, String message)
- static String  toString(Object o)
- static String  toString(Object o, String nullDefault)

詳細は[Javadocを参照](https://docs.oracle.com/javase/jp/7/api/java/util/Objects.html)してもらうとして、
私はrequireNonNullメソッドをよく利用しています。
いろいろなプロジェクトでメソッドの引数チェックの方法がマチマチだったのが、
このメソッドのおかげで引数チェック方法が統一できたのが画期的でした。
Lombokの@NonNullアノテーションでチェックすればいいじゃない？という話もありますが...(^^;

その昔、メソッドの引数がnullだった場合に、IllegalArgumentExceptionをスローすべきか、
NullPointerExceptionをスローすべきかという議論があったのですが、
java.util.ObjectsクラスのrequireNonNullメソッドがNullPointerExceptionをスローするので、
この議論はNullPointerExceptionの勝ちですかね (^^;

そして現在Java8がリリースされているわけですが、
さらに3つのメソッドが追加されました。

- static boolean isNull(Object obj)
- static boolean nonNull(Object obj)
- static <T> T   requireNonNull(T obj, Supplier<String> messageSupplier)

順当に利用頻度が高そうなメソッドが追加されています。
Supplierを第二引数に取るrequireNonNullメソッドの説明には「メッセージ・サプライヤを作成するコストが文字列メッセージを単に直接作成するコストよりも小さいことを考慮」
とありますが、サプライヤを生成するコストより文字列メッセージを生成するコストが上回るのはどのぐらいの文字列を生成しようとした場合ですかね？あとでちゃんと調べてみたいと思います。

まだApache-CommonsのObjectUilsクラスには及びませんが、
だいぶ使い勝手が良くなってきているのではないでしょうか。  
ぜひ、皆さんも使ってみてください。

メリークリスマス！


