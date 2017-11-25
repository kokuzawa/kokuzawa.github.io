---
title: "How to set the value in ValueChangeListener"
date: 2014-02-12T00:58:25+09:00
tags: [Java, JSF]
---
JSFには値が変更されたことをハンドリングするためのイベントとしてValueChangeEventが用意されています。
このイベントは、コンポーネントのValueChangeListenerによって発行されます。
ValueChangEventを利用してテキストフィールドに入力された値の検証を行っているのですが、
値が不正だった場合に、エラーを表示するのではなく正常な値を設定したい場合があります。

<!-- MORE -->

このとき、単純にValueChangeListenerの中でテキストフィールドのプロパティに値を設定しても、画面には反映されません。
これは、ValueChangeEventがJSFのライフサイクルのUpdate Model Valuesフェーズの前に呼ばれるからであり、
ValueChangeListenerで設定した値はUpdate Model Valuesフェーズで入力値によって上書きされてしまうからです。

これを解決するには、ValueChangeListenerにおいて、ちょっとした工夫が必要です。
以下にその例を示します。

``` java
public void valueChangeListener(ValueChangeEvent event)
{
    if (event.getPhaseId() == PhaseId.UPDATE_MODEL_VALUES) {
        property = "9:00";
    }
    else {
        String value = (String) event.getNewValue();
        if (false == value.matches("([0-9]{1,2}|1[0-9]|2[0-3]):[0-5][0-9]") {
            event.setPhaseId(PhaseId.UPDATE_MODEL_VALUES);
            event.queue();
        }
    }
}
```

この例では、時間の入力に対する検証を行い、不正な値が入力された場合にデフォルト値である「9:00」を設定しています。
値の検証をUpdate Model Valuesフェーズの前に実施し、値が不正である場合には、ValueChangeEventをUpdate Model Valuesフェーズにエンキューします。
このようにすることで、Update Model Valuesフェーズでもう一度ValueChangeListenerが呼び出されます。
二度ValueChangeListenerが呼ばれることになるので、イベントのフェーズIDを判定し、
Update Model Valuesフェーズで呼ばれた場合にデフォルト値を設定することで画面にその値を反映させます。

JSFは、ライフサイクルを知っていないと対処が難しい場合があるのがちょっといけてないですね...。
というわけで何かの参考になれば幸いです。

Enjoy !

