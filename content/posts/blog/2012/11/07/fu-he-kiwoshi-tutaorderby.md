---
title: "複合キーを使ったOrderBy"
date: 2012-11-07T00:56:00+09:00
tags: ["Java", "JPA"]
---
JPAで複合キークラスを使ってOrderByをやろうとしたら、メタデータの設定方法が分からなかったんだけど、分かってみれば簡単な話だった。

### 複合キーって何？
JPAのエンティティには複数の@Idアノテーションが付与できないので、DBのテーブルに主キーが複数ある場合はその主キーをひとまとめにしたクラスを作成する。エンティティ上では、@EmbeddedIdアノテーションを付与すれば、その作成クラスを主キーとして指定できる。これを複合キーといって、実際には下記のようなクラス構成となる。（EntityPK.javaが複合キー）

``` java
@Entity
public class Entity
{
    @EmbeddedId
    private EntityPK entityPK;

    public EntityPK getEntityPK()
    {
        return entityPK;
    }
}
```

``` java
@Embeddable
public class EntityPK implements Serializable
{
    @Column
    private Integer key1;

    @Column
    private Integer key2;
}
```

### CriteriaQueryでSQLを構築
この複合キーがあるエンティティに対してCriteriaQueryを使ってSQLを構築する。

``` java
final CriteriaBuilder cb = entityManager.getCriteriaBuilder();
final CriteriaQuery<Entity> query = cb.createQuery(Entity.class);
final Root<Entity> root = query.from(Entity.class);
query.select(root).orderBy(cb.asc(root.get(Entity_.entityPK)
    .get(EntityPK_.key1)));
```

ここでポイントとなるのが、key1でソートしようとした場合、いきなり
`cb.asc(root.get(EntityPK_.key1))` と指定はできないので、
`cb.asc(root.get(Entity_.entityPK).get(EntityPK_.key1))`
のように連鎖した呼び出しにする必要がある、ということ。
