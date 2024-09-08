# Добавление команд

Хоть генератор и поддерживает [SCM-функции](../base/scm\_functions.md), есть возможность добавления собственных команд. Сначала расскажу о них детальнее в теории.

### Принцип работы

Если говорить о работе команд, то следует запомнить, что команды — это не SCM-функции! Следовательно работать и компилироваться они будут иначе. Если сравнивать их работу с Sanny Buidler, то они напоминают работу [директивы](https://docs.sannybuilder.com/v/ru/coding/directives#usdinclude) `{$include}`: сначала создаётся отдельный файл с кодом, после чего они подключаются в нужном месте. Команды пишутся в похожем стиле, только вместо файлов код хранится в отдельном методе, а вместо директивы идёт вызов этого метода.

Главное отличие команды генератора от директивы SB в том, что команда может принимать параметры, что делает её намного гибче в плане разработки.

### Как создать команду?

Давайте рассмотрим примеры как они пишутся. Лучшим решением на текущий момент — использовать статические методы внутри класса **MAIN**. Функции пишутся как обычные методы класса. Напишем простой пример такой функции:

```csharp
public partial class MAIN : Thread {

    // наша новая функция и её реализация
    public static void destroy_actor( Actor hActor ) {
        hActor.destroy();
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN {

    public class TEST : Thread { // класс "TEST" внутри класса "MAIN"

        Actor tempActor;

        public override void START( LabelJump label ) {
            tempActor.create_random( 0.0, 0.0, 0.0 );
            // ...
            destroy_actor( tempActor ); // вызов нашей функции
            end_thread();
        }

    }

}
```

Давайте теперь напишем функции, которые регулируют трафик:

```csharp
public partial class MAIN : Thread {

    public static void __traffic_off() {
        set_vehicle_traffic_density_multiplier( 0.0 );
        set_ped_traffic_density_multiplier( 0.0 );
    }

    public static void __traffic_on() {
        set_vehicle_traffic_density_multiplier( 1.0 );
        set_ped_traffic_density_multiplier( 1.0 );
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        public override void START( LabelJump label ) { 
            __traffic_off();
            wait( 2000 );
            __traffic_on();
            end_thread();
        }

    }
}
```

При компиляции мы будем иметь такой код:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
01EB: set_traffic_density_multiplier_to 0.0
03DE: set_pedestrians_density_multiplier_to 0.0
0001: wait 2000 ms
01EB: set_traffic_density_multiplier_to 1.0
03DE: set_pedestrians_density_multiplier_to 1.0
004E: end_thread
```

Мы можем указать параметры методу, чтобы он стал более гибким:

```csharp
public partial class MAIN : Thread {

    public static void __toggle_traffic( double val ) {
        set_vehicle_traffic_density_multiplier( val );
        set_ped_traffic_density_multiplier( val );
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        public override void START( LabelJump label ) { 
            __toggle_traffic( 0.0 );
            wait( 2000 );
            __toggle_traffic( 1.0 );
            end_thread();
        }
		
    }
}
```

Использовать можно любые типы данных в качестве параметров. Но есть одно правило, которое следует помнить. Значение типа **float** не всегда правильно конвертируются. Вместо него нужно писать `double`. Это максимально приближает нас к синтаксису Sanny Builder и не нужно будет писать суффикс **F** для значений в C#.

Многие команды могут принимать значения переменных. В этом случае мы можем использовать типы данных, которые используются в генераторе. Следующий пример даёт возможность передавать литерал и переменную в команду.

```csharp
public partial class MAIN : Thread {

    public static void __toggle_traffic( Float density ) {
        set_vehicle_traffic_density_multiplier( density );
        set_ped_traffic_density_multiplier( density );
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        Float anyFloat;

        public override void START( LabelJump label ) { 
            __toggle_traffic( 0.0 );
            wait( 2000 );
            anyFloat.value = 1.0;
            __toggle_traffic( anyFloat );
            end_thread();
        }

    }
}
```

Кроме этого, мы можем писать некую логику, используя язык C#:

```csharp
public partial class MAIN : Thread {

    public static void __toggle_traffic( bool on ) {
        set_vehicle_traffic_density_multiplier( on ? 1.0 : 0.0 );
        set_ped_traffic_density_multiplier( on ? 1.0 : 0.0 );
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        public override void START( LabelJump label ) { 
            __toggle_traffic( false );
            wait( 2000 );
            __toggle_traffic( true );
            end_thread();
        }

    }
}
```

Как видим, в зависимости от параметра **on**, в команду будет подставляться нужное значение автоматически. Кроме стандартных типов, мы можем передавать и другие типы. Сложнее всего писать команду, которая требует любой транспорт:

```csharp
using GTA.Core; // <-- подключаем пространство имён, где хранится класс Vehicle<T>

public partial class MAIN : Thread {

    public static void __explode_car( Car hCar ) {
        hCar.explode();
    }

    public static void __explode_vehicle<T>( T hVehicle ) where T : Vehicle<T> {
        hVehicle.explode();
    }

    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        public override void START( LabelJump label ) { 
            __explode_car( Car.empty );
            //__explode_car( Boat.empty ); // <- ошибка: можно передать только транспорт типа "Car"

            __explode_vehicle( Boat.empty ); // можно передать любой транспорт
            __explode_vehicle( Car.empty ); // можно передать любой транспорт

            end_thread();
        }

    }
}
```

Обратите внимание, что некоторые классы имеют статическое свойство `empty`. Когда мы обращаемся к нему, вместо переменной будет указано значение **`-1`**. Ну и последний пример — реализация команд-условий. В этом случае нам нужно возвращать объект `Condition` в нашей команде:

```csharp
public partial class MAIN : Thread {

    public static Condition check_player( Player hPlayer ) {
        return hPlayer.is_autoaiming();
    }
	
    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public class TEST : Thread {

        public override void START( LabelJump label ) { 
            wait( 0 );
            jf( START, check_player( PlayerChar ) );
            end_thread();
        }

    }
}
```

Для проверки нескольких условий мы должны возвращать массив `Condition[]`:

```csharp
public partial class MAIN : Thread {

    public static Condition[] check_player() {
        return new Condition[] {
            PlayerChar.is_defined(),
            PlayerChar.is_controllable(),
            !PlayerChar.is_on_jetpack()
        };
    }
	
    public override void START( LabelJump label ) { }

}

// ...

public partial class MAIN : Thread {
    public partial class TEST : Thread {

        public override void START( LabelJump label ) { 
            wait( 0 );
            jf( START, check_player() );
            end_thread();
        }

    }
}
```

Последний пример будет иметь такой результат:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0001: wait 0 ms
00D6: if
0256:     player $2 defined
004D: jump_if_false @TEST
00D6: if
03EE:     player $2 controllable
004D: jump_if_false @TEST
00D6: if
8A0C: not player $2 on_jetpack
004D: jump_if_false @TEST
004E: end_thread
```

Неудобством подхода будет разве что дополнительный отступ.

{% hint style="danger" %}
Очень важно! Пользовательские команды должны возвращать типы **void**, **Condition** или **Condition\[]**. Использование других типов может привести к путанице и неправильной работе генератора!
{% endhint %}

### Недостатки

Пользовательские команды не лишены недостатков. Они отлично подходят в ситуациях, когда нам нужно объединить последовательный вызов базовых функций. Но совершенно уязвимы к декларации локальных переменных внутри себя! В основном это связано с тем, что при вызове команды она станет частью текущего скрипта (потока, миссии, внешного скрипта и SCM-функции) и все локальные переменные будут общими. Это может привести к ненужной перезаписи переменной или превышения лимита на эти же переменные.

Если в команде нужно использовать локальные переменные, то лучше указывать их как часть аргументов метода:

```csharp
public partial class MAIN : Thread {

    public static void __add_ammo_to_current_weapon( Actor hActor, Int ammo, Out<Int> tempWeapon ) {
        hActor.get_current_weapon( tempWeapon )
              .give_weapon( tempWeapon, ammo );
    }
    
}
```

В этом случае переменная **tempWeapon** будет передана из основного скрипта, не нарушая работы команды.

### Выводы

В целом, написание собственных команд сокращает код. Использовать его очень полезно. Некоторые вещи придётся тренировать, но результаты того стоят!
