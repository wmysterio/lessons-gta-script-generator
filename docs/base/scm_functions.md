# SCM-функции

Начиная с версии 7.3 генератор поддерживает SCM-функций. Их синтаксис практически тот же, что и синтаксис потоков. Для добавления функции нам нужно создать класс, который наследует класс `Function`. В отличии от потоков они должны завершать роботу командой `ret`.

```csharp
public class SUMM : Function {

    public override void START( LabelJump label ) {
        ret(); // 0AB2: ret 0
    }

}
```

Генератор не контролирует входные и выходные параметры. Всё это должен делать разработчик, поэтому важно знать как они работают.

Хорошо. Общие данные Вы уже знаете. Давайте теперь определимся какие параметры функция будет принимать. Допустим, это будет два числа. В этом случае мы можем декларировать локальные переменные:

```csharp
public class SUMM : Function {

    Int parameter0; // 0@
    Int parameter1; // 1@

    public override void START( LabelJump label ) {
        ret(); // 0AB2: ret 0
    }

}
```

Метод **ret** ничего не возвращает по умолчанию. Если нам нужны возвращаемые значения, то мы можем передать в этот метод нужные данные. Допустим, мы будем возвращать переменную, которая хранит результат сложения двух параметров:

```csharp
public class SUMM : Function {

    Int parameter0; // 0@
    Int parameter1; // 1@

    Int returned;   // 2@

    public override void START( LabelJump label ) {
        returned.value = parameter0;
        returned += parameter1;
        ret( returned ); // 0AB2: ret 1 2@
    }

}
```

Простая функция сложения двух чисел готова! Как нам теперь её вызывать? Да точно так же, как и в Sanny Builder: используем метод `call_scm_function`. Мы должны между символами `<` и `>` указать класс, который наследует класс **Function**. В нашем случае это **SUMM**. Также есть 1 обязательный параметр метода, в который надо передать количество аргументов функции, которые будут передаваться в качестве параметра. В нашем случае таких аргументов будет 2. Остальные параметры опциональны и зависят уже от требования самой функции. Вот пример вызова созданной выше функции:

```csharp
Int result = local( 25 );
call_scm_function<SUMM>( 2, 25, 10, result );
```

Запомнить параметры не всегда получается. Также синтаксис функции **call\_scm\_function** не дают точно определить какой конкретно тип нужен. Поэтому лучше этот вызов написать в [пользовательских командах](../tools/commands.md) или в [методах расширений](../tools/extensions.md). Вот пример как это сделать:

```csharp
public static void summ( Int parameter0, Int parameter1, Out<Int> result ) {
    call_scm_function<SUMM>( 2, parameter0, parameter1, result );
}
```

Согласитесь, что вызвать метод **summ** приятнее, хоть и нужно потратить немного времени на эту "обёртку". Обратите внимание, что я использовал тип `Out<Int>` для параметра **result**. Этот тип является неким фильтром, который позволяет передавать в него указанный тип (между символами `<` и `>`), а также гарантирует, что указанный параметр будет переменной, а не литералом.

{% hint style="danger" %}
Методы **call\_scm\_function** и **ret** могут принимать максимум 30 аргументов, а для GTA III и VC этот лимит составляет 14 аргументов (включает входные и выходные параметры)!
{% endhint %}

### Строки в SCM-функциях

Тип `Parameter` не разрешает передавать строки в качестве параметров. Если такая необходимость будет, то можно воспользоваться статический метод `GetParameters`. Он есть в классах **sString** и **vString**:

```csharp
public class TEST : Thread {

    vString myVString;
    sString mySString;

    public override void START( LabelJump label ) {
    
        myVString.value = "any name";
        mySString.value = "SSTRING";
        
        var vStrParameters = vString.GetParameters( myVString );
        var sStrParameters = sString.GetParameters( mySString );
        
        call_scm_function<ANY_FUNC_NAME1>( 4, vStrParameters[ 0 ], vStrParameters[ 1 ], vStrParameters[ 2 ], vStrParameters[ 3 ] );
        call_scm_function<ANY_FUNC_NAME2>( 2, sStrParameters[ 0 ], sStrParameters[ 1 ] );
    }

}

public class ANY_FUNC_NAME1 : Function {

    vString parameter0; // 0@-3@

    public override void START( LabelJump label ) {

        ret();
    }

}

public class ANY_FUNC_NAME2 : Function {

    sString parameter0; // 0@-1@

    public override void START( LabelJump label ) {

        ret();
    }

}
```

Этот метод возвращает **массив C#** типа **Int**. Дальше этот массив можно использовать для передачи и получения параметров, как это делается в Sanny Builder.

Процедура возврата строки из функции делается точно так же:

```csharp
public static void any_func_name1( Out<vString> result ) {
    var args1 = vString.GetParameters( result );
    call_scm_function<ANY_FUNC_NAME1>( 0, args1[ 0 ], args1[ 1 ], args1[ 2 ], args1[ 3 ] );
}
public static void any_func_name2( Out<sString> result ) {
    var args1 = sString.GetParameters( result );
    call_scm_function<ANY_FUNC_NAME2>( 0, args1[ 0 ], args1[ 1 ] );
}


public class ANY_FUNC_NAME1 : Function {

    vString parameter0; // 0@-3@ returned

    public override void START( LabelJump label ) {
        var args = vString.GetParameters( parameter0 );
        ret( args[ 0 ], args[ 1 ], args[ 2 ], args[ 3 ] );
    }

}

public class ANY_FUNC_NAME2 : Function {

    sString parameter0; // 0@-1@ returned

    public override void START( LabelJump label ) {
        var args = sString.GetParameters( parameter0 );
        ret( args[ 0 ], args[ 1 ] );
    }

}
```

### Условные SCM-функции

Вы можете использовать SCM-функции в качестве условий. За вызов такой функции отвечает метод `is_call_scm_function`. Он имеет те же параметры, что и **call\_scm\_function**, но может использоваться в блоке с условиями.

Для правильной работы требуется использовать команды `ret_true` и `ret_false` до вызова метода **ret**:

```csharp
public class MyСonditionalFunction : Function {

    public override void START( LabelJump label ) {
        and( PlayerActor.is_defined(), delegate {
            ret_true();
        }, delegate {
            ret_false();
        } );
        ret();
    }

}

public class TEST : Thread {

    public override void START( LabelJump label ) {
    
        and(
            is_call_scm_function<MyСonditionalFunction>( 0 )
        , delegate {
            show_formatted_text_box( "PLUS" );
        }, delegate {
            show_formatted_text_box( "MINUS" );
        } );
    
    }
    
}
```

Существуют также методы `ret_true_with_args`, `ret_false_with_args`, `ret_true_without_args` и `ret_false_without_args`. Они автоматически делают возврат, что полезно в некоторых ситуациях:

```csharp
public class MyСonditionalFunction : Function {

    public override void START( LabelJump label ) {
        and( PlayerActor.is_defined(), delegate {
            ret_true_without_args();
        }, delegate {
            ret_false_without_args();
        } );
    }

}
```

{% hint style="danger" %}
Опкод **0AB1** не может быть инвертирован коммандой **is\_call\_scm\_function**!
{% endhint %}

### Простой совет

Много функций и их "обёртки" несомненно будут накапливаться и могут дублировать имена друг-друга, что не позволит быстро вести поиск при вызове меню (`CTRL+Space`). Лучше всем функциям предоставить отдельное пространство имён:

```csharp
namespace MY_FUNTION_HERE {

    public class MY_FUNC : Function {

        public override void START( LabelJump label ) {
            ret();
        }

    }
    
    // здесь другие функции...
    
}
```

Тогда в "обёртке" доступ к **MYFUNC** будет осуществляться через пространство имён `MY_FUNTION_HERE`:

```csharp
public partial class MAIN : Thread {

    public static void my_func() {
        call_scm_function<MY_FUNTION_HERE.MY_FUNC>( 0 );
    }
    
}
```

Таким образом все классы для функций будут скрыты по умолчанию от основного кода, но доступны через `namespace`.
