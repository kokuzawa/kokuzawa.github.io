---
title: "Deprecated org.hibernate.ejb.HibernatePersistence"
date: 2015-01-27T23:46:21+09:00
tags: [Java, JPA, Hibernate] 
---
どのバージョンからなのかはわからないが、Hibernate-4.3.8.Finalではpersistence.xmlのproviderに記載するHibernateのプロバイダクラスがdeprecatedになっていた。

    <provider>org.hibernate.ejb.HibernatePersistence</provider>

上記を下記に置き換える必要がある。

    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

