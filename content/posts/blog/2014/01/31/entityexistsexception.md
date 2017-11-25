---
title: "EclipseLinkでの一意制約例外"
date: 2014-01-31T22:52:45+09:00
tags: [Java, JPA] 
---

[@kikutaro_](https://twitter.com/kikutaro_)さんのブログで、
EclipseLinkの一意制約例外について書かれた
「[EclipseLinkで一意制約の例外を拾ってみたけど…](http://kikutaro777.hatenablog.com/entry/2013/04/24/225439)」
の記事が Twitterで言及されていたので、ちょっと読んでたら`javax.persistence.EntityExistsException`がスローされないってあって、
あれ、そうだったけ？とちょっと疑問に思ったので調べてみた。

<!-- MORE -->

で、まあググると確かにスローされないって言っている人が沢山いたけど、
やっぱり自分で試してみないと気が済まないので、下記のようなコードを書いて試してみる。  
（コードにはlombokを適用してます）

**persistence.xml:**

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <persistence-unit name="testPU" transaction-type="RESOURCE_LOCAL">
        <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
        <class>entity.Account</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.apache.derby.jdbc.ClientDriver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:derby://localhost/test;create=true"/>
            <property name="javax.persistence.jdbc.user" value="APP"/>
            <property name="javax.persistence.jdbc.password" value="APP"/>
            <property name="eclipselink.ddl-generation" value="drop-and-create-tables"/>
        </properties>
    </persistence-unit>
</persistence>
```

**Account.java:**

``` java
@Entity
@NoArgsConstructor
@RequiredArgsConstructor
public class Account
{
    @Getter
    @NonNull
    @Id
    private Long id;

    @Getter @Setter
    @NonNull
    private String name;
}
```

**AccountTest.java:**

``` java
@Test(expected = EntityExistsException.class)
public void testPersist() throws Exception
{
    Account account = new Account(1L, "名前");
    em.persist(account);
}
```

accountテーブルには事前にid=1のデータを投入。  
この状態で実行すると下記のようなエラーがでる。確かに`EntityExistsException`はスローされない。

    javax.persistence.PersistenceException: Exception [EclipseLink-4002] (Eclipse Persistence Services - 2.5.1.v20130918-f2b9fc5): org.eclipse.persistence.exceptions.DatabaseException
    Internal Exception: java.sql.SQLIntegrityConstraintViolationException: このステートメントは、ユニークまたは主キー制約、または 'ACCOUNT' 上で定義された 'SQL140131234011701' によって識別されるユニーク索引において重複キー値の原因となる可能性があったため、打ち切られました。
    Error Code: -1
    Call: INSERT INTO account (ID, NAME) VALUES (?, ?)

うーん、仕様としては存在しているのだから、何かが間違っているのか...  
わからないので、EclipseLinkのコードを追ってみることに。

**org.eclipse.persistence.internal.jpa.EntityManagerImpl.java:**

``` java
try {
    getActivePersistenceContext(checkForTransaction(false)).registerNewObjectForPersist(entity, new IdentityHashMap());
} catch (RuntimeException exception) {
    if (exception instanceof ValidationException) {
        throw new EntityExistsException(exception.getLocalizedMessage(), exception);
    }
    throw exception;
}
```

コードを見る限り、`EntityManager#persist()`で`EntityExistsException`をスローしているように見える。
この`ValidationException`とは何なのか？どこから来たんだろう...。
というわけでここからどんどん辿って行くと、最終的にフラグが立っている場合にチェックしていることが分かる。
で、そのフラグがどこで立つのかをさらに辿って行くと、`eclipselink.validate-existence`というプロパティを参照している様子。

ああ、これはpersistence.xmlで設定できるやつだ、ということで下記をpersistence.xmlに設定する。

``` xml
<property name="eclipselink.validate-existence" value="true"/>
```

もう一回実行してみた。

    javax.persistence.EntityExistsException: 
    Exception Description: Cannot persist detached object [Account(id=1, name=名前)]. 
    Class> entity.Account Primary Key> 1

でた！確証を得るためにEclipseLinkのドキュメントを見てみる。

**[EclipseLink](http://www.eclipse.org/eclipselink/documentation/2.5/jpa/extensions/p_validate_existence.htm)**:

> Use the eclipselink.validate-existence persistence property configures to specify if EclipseLink should verify an object's existence on persist().

「eclipselink.validate-existenceプロパティを使用して、EclipseLinkがpersist()でオブジェクトの存在を検証するかどうかを指定します。」
ということなのでこれで合っているようだけど、デフォルトがfalseなので検証しないようだ。
ただ、ちょっと気になるのは使用方法の部分、

> EclipseLink will throw an error if a validated object is not in the persistence context.

not in the persistence context ってことは存在しない場合にエラーがスローされるの？  
この記述あってるのかなぁ？英語は得意じゃないので解釈が間違っているのかもしれない。
あ、persistence context に存在しないってことは、DBに存在していてfindしていなければエラーになるってことかな。

Enjoy !

