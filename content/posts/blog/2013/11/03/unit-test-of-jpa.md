---
title: "JUnitでJPAのテスト"
date: 2013-11-03T01:39:00+09:00
tags: [Java, JPA] 
---
JPAで作ったデータアクセスクラスのテストをJUnitで作る際に、EntityManagerの扱いが問題になってきます。
プロダクトコードだと、PersistenceUnitのtransactionTypeをJTAとするため、
この設定のままユニットテストを実行には、APサーバが起動している必要があります。
この状態だとJenkinsに食わせたりするときに色々大変だし、
そもそもテスト用にはデータベースを別にしたいので、設定をテスト時だけ書き直す必要が出てきて面倒です。
そこでプロダクトコードとテストコードで参照するPersistenceUnitを変更するようにしてみました。

## ファイル構成
ファイルは下記のように、プロダクトコードとテストコード用にそれぞれpersistence.xmlを用意します。

```
root
 └ src
   ├ main
   │  ├ java
   │  │  ├ Account.java
   │  │  └ AccountFacade.java
   │  └ resources
   │     └ META-INF
   │        └ persistence.xml
   └ test
      ├ java
      │  └ AccountFacadeTest.java
      └ resources
         └ META-INF
            └ persistence.xml
```

プロダクトコードのpersistence.xmlはこんな感じ。
transaction-typeをJTAにしているので、jta-data-sourceにAPサーバで設定したデータソース名を指定しています。

**persistence.xml:**

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <persistence-unit name="mytestPU" transaction-type="JTA">
        <jta-data-source>jdbc/mytest</jta-data-source>
        <properties>
            <property name="eclipselink.ddl-generation" value="create-tables"/>
        </properties>
    </persistence-unit>
</persistence>
```

テストコードのpersistence.xmlはこんな感じ。
プロダクトコードとは違い、APサーバが起動していなくてもユニットテストが実行できるように
transaction-typeをRESOURCE_LOCALにしています。

**persistence.xml:**

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <persistence-unit name="mytestTPU" transaction-type="RESOURCE_LOCAL">
        <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
        <class>org.mytest.entity.Account</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.postgresql.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:postgresql://localhost:5432/mytest_test"/>
            <property name="javax.persistence.jdbc.user" value="demo"/>
            <property name="javax.persistence.jdbc.password" value="demo"/>
            <property name="eclipselink.ddl-generation" value="drop-and-create-tables"/>
        </properties>
    </persistence-unit>
</persistence>
```

あとは下記のコードを用意します。

**Account.java:**

``` java
@Entity
public class Account
{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long accountId;
    private String email;
    ...
}
```

**AccountFacade.java:**

``` java
@RequestScoped
@Transactional
public class AccountFacade
{
    @PersistentContext(unitName = "mytestPU")
    private EntityManager em;

    public void persist(Account account)
    {
        Objects.requireNonNull(account);
        em.persist(account);
    }
}
```

**AccountFacadeTest.java:**

``` java
public class AccountFacadeTest
{
    private EntityManager em;
    private AccountFacade accountFacade = new AccountFacade();

    @Before
    public void setUp() throws Exception
    {
        em = Persistence.createEntityManagerFactory("mytestTPU").createEntityManager();
        accountFacade.em = em;
    }

    @After
    public void tearDown() throws Exception
    {
        em.close();
    }

    @Test
    public void testPersist() throws Exception
    {
        final Account account = new Account("xxxxx@hoge.com");

        em.getTransaction().begin();
        accountFacade.persist(account);
        em.getTransaction().commit();

        assert account.getAccountId() > 0;
    }
}
```

テストコードではmytestTPUからEntityManagerを生成します。
プロダクトコードでは@Transactionalでトランザクションを管理していますが、
テストコードでは@Transactionalでのトランザクション管理はできないので、
トランザクション制御のためのコードを書く必要があります。

Best Regards.

