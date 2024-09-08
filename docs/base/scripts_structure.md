# Структура скрипта

Как и в Sanny Builder, мы "разбиваем" наш код на отдельные части, именуемые как **Поток**, **Миссия** и **Внешний скрипт**. Чтобы генератор знал что где находится, используются классы `Thread`, `Mission`,`MissionCustom`и `External`. Примеры с использованием всех классов:

```csharp
using GTA;

public class MAIN : Thread {

    public override void START( LabelJump label ) {
        end_thread();
    }

}

public class MISS1 : Mission {

    public override void START( LabelJump label ) {
        // end_thread(); // НЕ НАДО!
    }

}

public class MISS12 : MissionCustom {

    public override void START( LabelJump label ) {
        end_thread(); // для MissionCustom НАДО!
    }

}

public class EXT : External {

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Если мы скомпилируем приложение (клавиша **F5**), то на выходе будем иметь следующий код (результат зависит от выбранной библиотеки и её версии, но смысл остаётся одним и тем же):

```
DEFINE OBJECTS 0

DEFINE MISSIONS 0

DEFINE EXTERNAL_SCRIPTS 0 // Use -1 in order not to compile AAA script

DEFINE UNKNOWN_EMPTY_SEGMENT 0

DEFINE UNKNOWN_THREADS_MEMORY 2048

//------------- THREAD MAIN ---------------
:MAIN
03A4: name_thread 'MAIN'
0180: set_on_mission_flag_to $409 // Note: your missions have to use the variable defined here
0111: set_wasted_busted_check 0
0004: $14 = 250 // $ = ? (int)
//===== EXTERNAL SCRIPT NOT FOUND =====\\
004E: end_thread
```

Как видим, мы написали небольшие участки кода и в итоге всё это создало код для **main.scm**, который очень нудно и долго набирать или копировать вручную в Sanny Builder. В этом и суть генератора — избавить нас от рутины.

Главный поток генерирует дополнительную информацию в начале — это переменная для состояния миссии, время задержки по умолчанию и регистрирует все внешние скрипты. Другие потоки создают только метку и дают название скрипту. Конец скриптов нужно вручную завершать командой `end_thread()`. Внешние скрипты и миссии имеют начальную метку с другим названием, чтобы предотвратить ошибку **Переход на нулевой оффсет**.

Таким образом Вы можете добавить нужное количество скриптов. В случае, если лимит на скрипты будет превышен или в ходе выполнения была допущена ошибка, генерация кода прекратится и будет показано сообщение о том, где была ошибка и её суть.

Запуск [потоков](labels\_jumps\_opcodes.md#zapusk-potokov), [миссий](missions.md#zapusk-missii) и [внешних скриптов](external\_scipts.md) будет рассмотрено в отдельных статьях.

{% hint style="danger" %}
Класс **External** доступен только в библиотеке **GTA.SA**!
{% endhint %}

{% hint style="danger" %}
Первым скриптом будет выполнятся класс с именем **MAIN**!
{% endhint %}

