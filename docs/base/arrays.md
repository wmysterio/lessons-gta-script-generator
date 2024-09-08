# Массивы

Скриптовый движок для GTA San Andreas позволяет использовать массивы. Эту возможность я добавил и в генератор. Я рекомендую научится ими пользоваться, так как это существенно ускоряет разработку миссий и позволяет сократить количество ручного кода при работе с множеством данных.

### Инициализация

Начиная с версии 7.0 встроен упрощённый синтаксис инициализации массивов, который будет самостоятельно определять контекст и начало массива. Нам останется только указать его размер:

```csharp
public class TEST : Thread {

    static Array<Car> carArray = 10; // $2000 - $2009
    Array<Actor> pedArray = 10; // 0@ - 9@

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Здесь: **10** — это количество элементов в массиве. Модификатор `static` создаёт массив глобальных переменных, а без него будет создан массив локальных переменных. Между символами `<` и `>` указывается тип данных (генератор поддерживает практически все типы). А **carArray** и **pedArray** — имена массивов. Также поддерживается старый синтаксис инициализации, он выглядит так:

```csharp
public class TEST : Thread {

    Array<Car> carArray = global_array( 10 ); // $2000 - $2009
    Array<Actor> pedArray = local_array( 10 ); // 0@ - 9@

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Метод `global_array` будет создавать глобальный массив, а `local_array` — локальный массив. При этом модификатор **static** уже не будет определять контекст. Его, как и в случае с обычными переменными, можно использовать внутри методов. Начальный индекс переменной вычисляется автоматически для обоих вариантов. Также генератор делает нужные отступы, если тип данных это требует (типы **vString**, **sString**).

Если нужно определить массив самостоятельно, то используется перегрузка методов **global\_array** и **local\_array**, передав первым параметром индекс, который будет первым элементом:

```csharp
public class TEST : Thread {

    Array<Car> carArray = global_array( 4000, 10 ); // $4000 - $4009
    Array<Actor> pedArray = local_array( 10, 10 ); // 10@ - 19@

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

### Установка значений

Мы можем обращаться к массиву по индексу (индекс указывается в квадратных скобках) элемента и давать команды, которые есть в указанного типа данных:

```csharp
public class TEST : Thread {

    Array<Actor> pedArray; // 0@-9@

    public override void START( LabelJump label ) {

        pedArray[ 0 ].create_random( 0.0, 1.0, 2.0 );
        pedArray[ 1 ].create_random( 1.0, 1.0, 2.0 );
        pedArray[ 2 ].create_random( 2.0, 1.0, 2.0 );
        // ...
        pedArray[ 9 ].create_random( 9.0, 1.0, 2.0 );

        end_thread();
    }

}
```

Если тип требует более одной переменной, то генератор это будет учитывать и делать нужные отступы. Это требуют только два типа данных — **sString** и **vString**:

```csharp
public class TEST : Thread {

    Array<Actor> pedArray = 2; // 0@-1@

    public override void START( LabelJump label ) {

        pedArray[ 0 ].create_random( 0.0, 1.0, 2.0 );
        pedArray[ 1 ].create_random( 1.0, 1.0, 2.0 );

        Array<vString> stringArray = local_array( 2, 3 ); // 2@,3@,4@,5@    6@,7@,8@,9@    10@,11@,12@,13@
        stringArray[ 0 ].value = "str 0";
        stringArray[ 1 ].value = "str 1";
        stringArray[ 2 ].value = "str 2";

        end_thread();
    }

}
```

Мы увидим смещение в результате:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0376: 0@ = create_random_actor_at 0.0 1.0 2.0
0376: 1@ = create_random_actor_at 1.0 1.0 2.0
06D2: 2@v = "str 0" // @v = ? (vString)
06D2: 6@v = "str 1" // @v = ? (vString)
06D2: 10@v = "str 2" // @v = ? (vString)
004E: end_thread
```

Заметно, что никаких массивов не используется на выходе. Здесь активно внедрён трюк с переменными, который позволяет объединять переменные в массив.

### Метод each

Мы можем вызвать метод `each`, который будет делать указанное действие с каждым элементом массива:

```csharp
public class TEST : Thread {

    Int index;
    Array<sString> strArray = 5;

    public override void START( LabelJump label ) {

        strArray.each( index, elem => {
            elem.value = sString.DUMMY;
        } );

        end_thread();

    }

}
```

Внутри этой команды "вшит" цикл, который требует переменную-счётчик. Параметр `elem` нужно для того, чтобы работать с элементом массива. Мы можем использовать и ручной цикл, передав переменную счётчик в индекс массива:

```csharp
public class TEST : Thread {

    Int index;
    Array<sString> strArray = 5;

    public override void START( LabelJump label ) {

        // ручной перебор массива
        to( index, 0, strArray.count - 1, delegate {
            strArray[ index ].value = "DUMMY";
        } );

        strArray.each( index, elem => {
            elem.value = sString.DUMMY;
        } );

        end_thread();

    }

}
```

Оба варианта будут генерировать одно и тоже. Только второй делает это быстрее. Вот результат:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'

int 0@
for 0@ = 0 to 4
05AA: 1@(0@,5s) = 'DUMMY' // @s = ? (sString)
end


int 0@
for 0@ = 0 to 4
05AA: 1@(0@,5s) = 'DUMMY' // @s = ? (sString)
end

004E: end_thread
```

{% hint style="warning" %}
В методе **each** нельзя использовать команды **break** и **continue**!
{% endhint %}

