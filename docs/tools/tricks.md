# Трюки

В этой теме рассмотрены различные вспомогательные механизмы для работы с кодом.

### Упрощения в условиях

Код скриптов можно упрощать, используя сам язык C#. Например, если при выполнении условия надо совершить только одну команду, которая соответствует делегату **System.Action**, мы можем написать только ссылку на метод. До:

```csharp
and(
   PlayerChar.is_controllable(),
   is_key_pressed( VKeys.F4 )
   // ...
, delegate { show_save_screen(); } );
```

После:

```csharp
and(
   PlayerChar.is_controllable(),
   is_key_pressed( VKeys.F4 )
   // ...
, show_save_screen ); // -13 символов: delegate { (); }
```

Это тоже касается для блока **else**:

```csharp
and(
   PlayerChar.is_controllable(),
   is_key_pressed( VKeys.F4 )
   // ...
, show_save_screen, show_credits ); // если условия не выполняются, то запускаем титры
```

Особенно это полезно при вызове методов `@break` и `@continue`.

### Модели поездов

Каждый тип поезда требует загрузки определённых моделей. Чтобы их не искать, в класс **Train** встроен статический метод `GetModelsByType`, который возвращает массив с моделями для конкретного типа поезда:

```csharp
public sealed class TEST : Thread {

    Train myTrain;

    public override void START( LabelJump label ) {

        var trainType = TrainType.FREIGHT_FREIFLAT4;

        var trainModels = Train.GetModelsByType( trainType ); // получаем нужные модели вагонов и поезда

        load_requested_models( trainModels );
        myTrain.create( 0.0, 0.0, 0.0, trainType, false ).remove_references();
        destroy_model( trainModels );

        end_thread();
    }

}
```

Результат:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0247: load_model 537
0247: load_model 569
038B: load_requested_models
06D8: 0@ = create_train_at 0.0 0.0 0.0 type 0 direction 0
07BE: remove_references_to_train 0@
0249: release_model 537
0249: release_model 569
004E: end_thread
```

### Небезопасный код

Вы можете добавить любой код в скрипт. Для этого есть метод `unsafe_code`. Как и метод **hex**, он не проверяет содержимое и печатает указанную строку в том виде, в котором её изначально указали. Этот подход позволяет добавлять код, который не имеется в арсенале генератора. Пример использования:

```csharp
unsafe_code( @"

0494: get_joystick 0 data_to 10@ 11@ 12@ 12@
10@ += 50
12@ -= 13
// ...

" );
```

Этот метод отлично справляется с кодом, в котором не используются глобальные переменные или массивы, нет работы с миссиями, потоками и внешними скриптами. Если переданный скрипт содержит одну их вышеупомянутых сущностей, то придётся делать делать замены в строке, чтобы SB не перезаписал значения переменных, используемых в генераторе. Собственно последнее как раз заставило меня назвать метод **unsafe** (небезопасный).

#### Обработка глобальных переменных

Если добавленный код содержит глобальную переменную, то вот рекомендации как избежать возможных плохих последствий:

1. Найдите все глобальные переменные, которые есть в скрипте.
2. Отсортируйте их по длине имени. Некоторые имена могут частично совпадать с другими переменными.
3. Декларируйте эти глобальные переменные в классе скрипта.
4. Сделайте замену всех переменных с исходного варианта на новые переменные с класса.

Предположим, что у нас есть такой код:

```csharp
partial class MAIN : Thread {

    class TEST : Thread {
    
        public override void START( LabelJump label ) {
            unsafe_code( @"

$1500 = 2.5  // <--
$1501 = 0.2  // <--
$1502 = -9.8 // <--

" );
        } 
    
    }

}
```

Переменные `$1500` , `$1501`, и `$1502` могут вызвать сбой. Тогда пользуясь инструкцией выше, сделаем замены строки на новые декларированные переменные. Пример:

```csharp
partial class MAIN : Thread {

    static Float _1500, _1501, _1502;

    class TEST : Thread {
    
        public override void START( LabelJump label ) {
            unsafe_code( @"

" + _1500 + @" = 2.5  // <--
" + _1501 + @" = 0.2  // <--
" + _1502 + @" = -9.8 // <--

" );
        } 
    
    }

}
```

Делать замену можно в любом текстовом редакторе и любым способом. Название `$VARNAME` заменяем на `" + VARNAME + @"`, где **VARNAME** — имя декларированной переменной в классе. Процедуру повторяем, пока в коде не будет глобальных переменных.

{% hint style="success" %}
Индексы **$ON\_MISSION**, **$PLAYER\_ACTOR**, **$PLAYER\_GROUP** и **$PLAYER\_CHAR** полностью совпадают с индексами в генераторе, который эти переменные декларирует самостоятельно. Поэтому нет необходимости в их замене. Если это всё таки нужно сделать, то достаточно использовать декларированные имена **OnMission**, **PlayerActor**, **PlayerGroup** и **PlayerChar** соответственно, а не создавать новые переменные.
{% endhint %}

#### Обработка массивов

Если в коде встречается массив, то тактика коррекции кода немного отличается. Мы по-прежнему должны декларировать массив, указав его конкретный размер; но для корректной замены нужно декларировать ещё одну переменную, которая будет иметь индекс первого элемента массива. Получить индекс мы можем используя метод `Int.IndexOf( _1600 )`, где **\_1600** — декларируемый массив или другая переменная с контекстом. Для примера возьмем следующий код:

```csharp
partial class MAIN : Thread {

    class TEST : Thread {
    
        public override void START( LabelJump label ) {
            unsafe_code( @"

$1600[0] = 2 // <--
$1600[1] = 0 // <--
$1600[2] = 0 // <--

" );
        } 
    
    }

}
```

Алгоритм действий таков:

1. Ищем размер массива в коде и его тип. В нашем случае размер равен 3-м, а тип — **Int**;
2. Декларируем глобальный массив в классе **MAIN**;
3. Декларируем новую переменную внутри метода **START**, используя методы `global` и `IndexOf`. Это нужно для того, чтобы генератор успел инициализировать массив перед тем как получать индекс.
4. Делаем замену _**имени массива**_ в исходном варианте на декларированную переменную (не массив).

Как всё получится в итоге:

```csharp
partial class MAIN {

    static Array<Int> _1600 = 3;

    public class TEST : Thread {

        public override void START( LabelJump label ) {

            // создаём переменную, взяв индекс
            // первого элемента массива _1600
            Int tmp1600 = global( Int.IndexOf( _1600 ) );

            unsafe_code( @"

" + tmp1600 + @"[0] = 2 // <--
" + tmp1600 + @"[1] = 0 // <--
" + tmp1600 + @"[2] = 0 // <--

" );
        }

    }
    
}
```

Процедуру повторяем, пока в коде не будет глобальных массивов.

{% hint style="success" %}
Статический метод **IndexOf** доступен не только в классе "Int", но и в большинстве других. Проще будет привыкнуть именно к "Int", так как метод возвращает целое число.
{% endhint %}

#### Обработка потоков, миссий и внешних скриптов

Если внутри текста будет вызов потока, то надо убедиться, что в проекте нет класса с таким именем. Тогда можно обойтись без замен. Если поток в коде будет вызываться в нескольких скриптах, то лучше этот поток реализовать в коде, чтобы генератор о нём знал и не создавал дубликатов. Для этого мы должны изолировать вызов потока от остального текста:

```csharp
partial class MAIN {

    class TEST1 : Thread {
    
        public override void START( LabelJump label ) {
            unsafe_code( @"wait 0");
            
            create_thread<TEST2>(); // запускаем поток
            
            unsafe_code( @"// ...    
end_thread
");
        }
        
    }
    
    class TEST2 : Thread {

        public override void START( LabelJump label ) {
            unsafe_code( @"
            
wait 0
// ...    
end_thread    
 
");
        }
        
    }

}
```

Аналогично всё делается с миссиями и внешними скриптами. При этом стоит учитывать, что надо делать замену для всех методов, которые связаны с этими сущностями.
