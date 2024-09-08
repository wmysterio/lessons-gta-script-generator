# Миссии

### Запуск миссий

Миссии имеют два варианта генерации: **автоматический** (`Mission`) и **персональный** (`MissionCustom`). Первый вариант генерирует код, который уже имеет реализацию и требует только написать условия провала и победы. Персональный позволяет написать миссию самостоятельно с нуля. Для начала научимся запускать миссию. Для этого используется команда `start_mission`:

```csharp
public partial class MAIN : Thread {

    public override void START( LabelJump label ) {

        start_mission<MISS1>();
        end_thread();

    }

}

public class MISS1 : MissionCustom {

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Номер миссии будет взят автоматически. Нам достаточно передать методу **start\_mission** между символами `<` и `>` тип данных, который наследует **Mission** или **MissionCustom**. Код выше будет генерироваться вот так:

```
DEFINE OBJECTS 0

DEFINE MISSIONS 1
DEFINE MISSION 0 AT @MISS1_ERROR_SKIP			// 0

DEFINE EXTERNAL_SCRIPTS 0 // Use -1 in order not to compile AAA script

DEFINE UNKNOWN_EMPTY_SEGMENT 0

DEFINE UNKNOWN_THREADS_MEMORY 2048

//------------- THREAD MAIN ---------------
:MAIN
03A4: name_thread 'MAIN'

0417: start_mission 0
004E: end_thread

//------------- MISSION: 0 ---------------
:MISS1_ERROR_SKIP
03A4: name_thread 'MISS1'

:MISS1
004E: end_thread
```

### Принцип работы

**Персональный** вариант мы рассматривать не будем, так как принцип работы похож на обычные потоки. Класс **Mission** имеет более мощный функционал, поэтому рационально будет остановится детальнее на нём.

Класс позволяет генерировать уже готовый код миссии, который остаётся только расширить своим функционалом. Генератор создаст метки провала, победы и очистки миссии от мусора. Доступ к этим меткам осуществляется с помощью свойств `OnPassed`, `OnFailed` и `OnClear`, в которые надо указать анонимную функцию с нашим кодом:

```csharp
public class TEST : Mission {

    public override void START( LabelJump label ) {

        OnPassed = delegate { /* успех */ };

        OnFailed = delegate { /* провал */ };

        OnClear =  delegate { /* очистка */ };
		
    }

}
```

Класс **Mission** добавляет специальные команды, которые помогут в нужный момент сделать переход на метки провала и успеха, а также осуществить **gosub** на метку очистки. Это методы `jump_passed`, `jump_failed` и `gosub_clear`. Давайте напишем код небольшой миссии с использованием всего функционала:

```csharp
public class TEST : Mission {

    Int index, counter;
    Float x;
    Array<Actor> enemyArray = 9;
    Array<Marker> enemyMarkerArray = 9;

    public override void START( LabelJump label ) {

        x.value = 0.0;
        enemyArray.each( index, actor => {
            actor.create_random( x, 1400.0, 13.0 );
            enemyMarkerArray[ index ].create_above_actor( actor );
            x += 1.2;
        } );

        Cycle += LOOP;

        OnPassed = delegate { show_text_styled( "M_PASSD", 5000, 1 ); };

        OnFailed = delegate { show_text_styled( "M_FAIL", 5000, 1 ); };

        OnClear = delegate {
            Comment = "clear all markers and actors";

            to( index, 0, enemyMarkerArray.count - 1, delegate {

                and( enemyMarkerArray[ index ].is_enabled(), delegate {
                    enemyMarkerArray[ index ].disable();
                } );

                and( enemyArray[ index ].is_defined(), delegate {
                    enemyArray[ index ].remove_references().destroy();
                } );

            } );

        };

    }

    private void LOOP() {
        wait( 0 );

        counter.value = 0;

        enemyArray.each( index, actor => {

            and( actor.is_dead(), delegate {
                counter += 1;

                and( enemyMarkerArray[ index ].is_enabled(), delegate {
                    enemyMarkerArray[ index ].disable();
                } );

            } );

        } );

        and( counter >= enemyArray.count - 1, delegate {
            jump_passed();
        } );

    }

}
```

Методы **OnPassed**, **OnFailed** и **OnClear** можно написать в любом месте кода миссии, они обрабатываются отдельно. Я предпочитаю писать их в конце метода **START**. Вот такой код получится в итоге:

```
//------------- MISSION: 0 ---------------
:TEST_ERROR_SKIP
03A4: name_thread 'TEST'
0317: increment_mission_attempts
0004: $409 = 1 // $ = ? (int)
0050: gosub @TEST
00D6: if
0112:     wasted_or_busted // mission only
004D: jump_if_false @TEST_END
0050: gosub @TEST_FAILED

:TEST_END
0004: $409 = 0 // $ = ? (int)
00D8: mission_cleanup
004E: end_thread

:TEST
0007: 36@ = 0.0 // @ = ? (float)

int 34@
for 34@ = 0 to 8
0376: 37@(34@,9i) = create_random_actor_at 36@ 1400.0 13.0
0187: 46@(34@,9i) = create_marker_above_actor 37@(34@,9i)
000B: 36@ += 1.2 // @ += ? (float)
end


:TEST_CYCLE_0
0001: wait 0 ms
0006: 35@ = 0 // @ = ? (int)

int 34@
for 34@ = 0 to 8
00D6: if
0118:     actor 37@(34@,9i) dead
then
000A: 35@ += 1 // @ += ? (int)
00D6: if
075C:     marker 46@(34@,9i) enabled
then
0164: disable_marker 46@(34@,9i)
end
end
end

00D6: if
0029:     35@ >= 8 // @ >= ? (int)
then
0002: jump @TEST_PASSED
end
0002: jump @TEST_CYCLE_0

:TEST_PASSED
0595: mission_complete
0050: gosub @TEST_CLEAR
00BA: show_text_styled GXT 'M_PASSD' time 5000 style 1
0051: return

:TEST_FAILED
045C: abort_mission
0050: gosub @TEST_CLEAR
00BA: show_text_styled GXT 'M_FAIL' time 5000 style 1
0051: return

:TEST_CLEAR
// clear all markers and actors

int 34@
for 34@ = 0 to 8
00D6: if
075C:     marker 46@(34@,9i) enabled
then
0164: disable_marker 46@(34@,9i)
end
00D6: if
056D:     actor 37@(34@,9i) defined
then
01C2: remove_references_to_actor 37@(34@,9i) // Like turning an actor into a random pedestrian
009B: destroy_actor 37@(34@,9i)
end
end

0051: return
```

Часто будут возникать ситуации, когда нужно написать собственный код провала. В помощь нам придут команды прыжков на метку провала и победы, а также переход с возвратом на метку очистки миссии от мусора. Рассмотрим пример реализации такой ситуации:

```csharp
public class TEST : Mission {

    Checkpoint checkpoint;

    public override void START( LabelJump label ) {

        checkpoint.create( 140.0, 1440.0, 13.0 );

        wait( PlayerActor.is_near_point_3d_on_foot( 1, 140.0, 1440.0, 13.0, 3.0, 3.0, 3.0 ) );

        and( !PlayerChar.is_money_greater( 4000 ), delegate {
            jump_failed(); // прыжок на метку провала
        } );

        and( PlayerActor.is_current_weapon( WeaponNumber.AK47 ), delegate {
            Jump += CUSTOM_FAILED_CODE; // прыжок на собственную метку провала
        } );

        jump_passed(); // прыжок на метку победы

        OnPassed = delegate { show_text_styled( "M_PASSD", 5000, 1 ); };
        OnFailed = delegate { show_text_styled( "M_FAIL", 5000, 1 ); };
        OnClear = delegate { checkpoint.disable(); };

    }

    private void CUSTOM_FAILED_CODE( LabelJump label ) {
        show_text_highpriority( "GXTNAME", 5000, 1 );
        jump_failed(); // прыжок на метку провала
    }

    /* или такой вариант:
    private void CUSTOM_FAILED_CODE( LabelJump label ) {
        gosub_clear(); // переход на метку чистки, с возвратом
        show_text_highpriority( "GXTNAME", 5000, 1 );
        show_text_styled( "M_FAIL", 5000, 1 );
        @return(); // делаем возврат к коду завершения миссии
    }
    */

}
```

Если использовать персональные блоки для провала или успеха, то перед этими метками команду **jump\_passed** использовать обязательно! Также обязательным условием является завершать свои блоки командой `@return`. Вот такой результат получится:

```
//------------- MISSION: 0 ---------------
:TEST_ERROR_SKIP
03A4: name_thread 'TEST'
0317: increment_mission_attempts
0004: $409 = 1 // $ = ? (int)
0050: gosub @TEST
00D6: if
0112:     wasted_or_busted // mission only
004D: jump_if_false @TEST_END
0050: gosub @TEST_FAILED

:TEST_END
0004: $409 = 0 // $ = ? (int)
00D8: mission_cleanup
004E: end_thread

:TEST
018A: 34@ = create_checkpoint_at 140.0 1440.0 13.0

:TEST_JF_0
0001: wait 0 ms
00D6: if
00FF:     actor $3 sphere 1 in_sphere 140.0 1440.0 13.0 radius 3.0 3.0 3.0 on_foot
004D: jump_if_false @TEST_JF_0
00D6: if
810A: not player $2 money > 4000
then
0002: jump @TEST_FAILED
end
00D6: if
02D8:     actor $3 current_weapon == 30
then
0002: jump @TEST_LABEL_0
end
0002: jump @TEST_PASSED

:TEST_PASSED
0595: mission_complete
0050: gosub @TEST_CLEAR
00BA: show_text_styled GXT 'M_PASSD' time 5000 style 1
0051: return

:TEST_FAILED
045C: abort_mission
0050: gosub @TEST_CLEAR
00BA: show_text_styled GXT 'M_FAIL' time 5000 style 1
0051: return

:TEST_CLEAR
0164: disable_marker 34@
0051: return

:TEST_LABEL_0
00BC: show_text_highpriority GXT 'GXTNAME' time 5000 flag 1
0002: jump @TEST_FAILED
```
