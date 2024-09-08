# Таблицы переходов

В скриптах для GTA San Andreas есть возможность использовать таблицы переходов. Опкод имеет громадный размер, и постоянно редактировать метки, добавление или удаление блоков было очень нудным и хлопотным делом. Генератор скриптов существенно упрощает написание подобного рода код. Вот так можно написать код таблицы переходов:

```csharp
public class TEST : Thread {

    Int anyInt;

    public override void START( LabelJump label ) {

        anyInt.random( 0, 10 );

        jump_table( anyInt, jt => {

            // тут будут создаваться метки для переходов

        } );

        end_thread();

    }

}
```

Здесь `jt` — это объект, который позволит нам создавать переходы и ассоциировать их с числом. Ассоциация происходит с помощью метода `label`. Пример использования:

```csharp
public class TEST : Thread {

    Int anyInt;

    public override void START( LabelJump label ) {

        anyInt.random( 0, 10 );

        jump_table( anyInt, jt => {

            jt.label( 0, label => {
                Comment = "case #0";
            } );

            jt.label( 1, label => {
                Comment = "case #1";
            } );

            jt.label( 2, delegate {
                Comment = "case #2";
            } );

            // или использовать цепочку:
            jt.label( 3, label => {
                Comment = "case #3";
            } ).label( 4, label => {
                Comment = "case #4";
            } ).label( 5, label => {
                Comment = "case #5";
            } ).label( 6, label => {
                Comment = "case #6";
            } ).label( 7, delegate {
                Comment = "case #7";
            } ).label( 8, delegate {
                Comment = "case #8";
            } ).label( 9, delegate {
                Comment = "case #9";
            } ).label( 10, delegate {
                Comment = "case #10";
            } );

        } );

        end_thread();

    }

}
```

Если количество блоков превысит максимальный размер, то автоматически будет создано продолжение таблицы переходов, как это показано ниже, в результате:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0209: 0@ = random_int_in_ranges 0 10
0871: init_jump_table 0@ total_jumps 11 default_jump 0 @TEST_SWITCH_0_CASE_END jumps 0 @TEST_SWITCH_0_CASE_0 1 @TEST_SWITCH_0_CASE_1 2 @TEST_SWITCH_0_CASE_2 3 @TEST_SWITCH_0_CASE_3 4 @TEST_SWITCH_0_CASE_4 5 @TEST_SWITCH_0_CASE_5 6 @TEST_SWITCH_0_CASE_6
0872: jump_table_jumps 7 @TEST_SWITCH_0_CASE_7 8 @TEST_SWITCH_0_CASE_8 9 @TEST_SWITCH_0_CASE_9 10 @TEST_SWITCH_0_CASE_10 -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_0
// case #0

:TEST_SWITCH_0_CASE_1
// case #1

:TEST_SWITCH_0_CASE_2
// case #2

:TEST_SWITCH_0_CASE_3
// case #3

:TEST_SWITCH_0_CASE_4
// case #4

:TEST_SWITCH_0_CASE_5
// case #5

:TEST_SWITCH_0_CASE_6
// case #6

:TEST_SWITCH_0_CASE_7
// case #7

:TEST_SWITCH_0_CASE_8
// case #8

:TEST_SWITCH_0_CASE_9
// case #9

:TEST_SWITCH_0_CASE_10
// case #10

:TEST_SWITCH_0_CASE_END
004E: end_thread
```

В примере я использовал делегат. Он нужен в том случае, если нет необходимости делать переходы на метки, которые генерируются методом `label`. Если эту метку нужно использовать (например: при условиях), то пользуемся анонимным методом с сигнатурой `label =>` вместо **delegate**. Объект **jt** также имеет свойство `EndLabel`, которое хранит метку выхода с таблицы. Она генерируется в конце всех блоков. Пример:

```csharp
public class TEST : Thread {

    Int anyInt;

    public override void START( LabelJump label ) {

        anyInt.random( 0, 10 );

        jump_table( anyInt, jt => {

            jt.label( 0, label => {
                wait( 0 );
                and( label, PlayerChar.is_defined() ); // переход на текущую CASE-метку, если условие не выполняется
                Comment = "case #0";
                jump( jt.EndLabel ); // переход в конец таблицы
            } );

            jt.label( 1, label => {
                wait( 0 );
                Comment = "case #1";
                jump( label ); // переход на текущую CASE-метку
            } );

            jt.label( 2, delegate {
                Comment = "case #2";
                jump( jt.EndLabel ); // переход в конец таблицы
            } );

        } );

        // здесь будет метка выхода с таблицы
        wait( 2500 );
        end_thread();

    }

}
```

Метку конца таблицы также можно получить с объекта **label** в данном примере:

```csharp
//...
jt.label( 0, label => {
    Comment = "case #0";
    jump( label.EndLabel ); // переход в конец таблицы
} );
//...
```

Результат:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0209: 0@ = random_int_in_ranges 0 10
0871: init_jump_table 0@ total_jumps 3 default_jump 0 @TEST_SWITCH_0_CASE_END jumps 0 @TEST_SWITCH_0_CASE_0 1 @TEST_SWITCH_0_CASE_1 2 @TEST_SWITCH_0_CASE_2 -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_0
0001: wait 0 ms
00D6: if
0256:     player $2 defined
004D: jump_if_false @TEST_SWITCH_0_CASE_0
// case #0
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_1
0001: wait 0 ms
// case #1
0002: jump @TEST_SWITCH_0_CASE_1

:TEST_SWITCH_0_CASE_2
// case #2
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_END
0001: wait 2500 ms
004E: end_thread
```

Если значения "кейсов" возрастают от нуля каждый раз на единицу, то можно воспользоваться событием `Auto`. Вместо анонимных функций более компактный вид имеет следующий код:

```csharp
public class TEST : Thread {

    Int anyInt;

    public override void START( LabelJump label ) {

        anyInt.random( 0, 10 );

        jump_table( anyInt, jt => {

            // В столбик добавляем новый обработчик каждого кейса

            jt.Auto += case_0; // anyInt == 0
            jt.Auto += case_1; // anyInt == 1
            jt.Auto += case_2; // anyInt == 2
            jt.Auto += case_3; // anyInt == 3

            void case_0( LabelCase label ) { // локальная функция внутри метода "jump_table"
                Comment = "case #0";
                jump( jt.EndLabel );
            }

            void case_1( LabelCase label ) { // локальная функция внутри метода "jump_table"
                Comment = "case #1";
                jump( jt.EndLabel );
            }

            void case_2( LabelCase label ) { // локальная функция внутри метода "jump_table"
                Comment = "case #2";
                jump( jt.EndLabel );
            }

            void case_3( LabelCase label ) { // локальная функция внутри метода "jump_table"
                Comment = "case #3";
                jump( jt.EndLabel );
            }

        } );

        // здесь будет метка выхода с таблицы
        wait( 2500 );
        end_thread();

    }

}
```

Так, как у нас нет возможности использовать объект **jt** вне метода **START** (в этом примере), то все функции нужно сделать локальными. Также они позволяют не разбрасывать код по всему файлу. Вот результат:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
0209: 0@ = random_int_in_ranges 0 10
0871: init_jump_table 0@ total_jumps 4 default_jump 0 @TEST_SWITCH_0_CASE_END jumps 0 @TEST_SWITCH_0_CASE_0 1 @TEST_SWITCH_0_CASE_1 2 @TEST_SWITCH_0_CASE_2 3 @TEST_SWITCH_0_CASE_3 -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END -1 @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_0
// case #0
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_1
// case #1
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_2
// case #2
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_3
// case #3
0002: jump @TEST_SWITCH_0_CASE_END

:TEST_SWITCH_0_CASE_END
0001: wait 2500 ms
004E: end_thread
```

Если не использовать локальные функции, то доступ к метке конца таблицы можно получить с объекта **LabelCase**, воспользовавшись свойством `EndJumpTable`:

```csharp
public class TEST : Thread {

    Int anyInt;

    public override void START( LabelJump label ) {

        anyInt.random( 0, 10 );

        jump_table( anyInt, jt => {

            jt.Auto += case_0; // anyInt == 0
            jt.Auto += case_1; // anyInt == 1
            jt.Auto += case_2; // anyInt == 2
            jt.Auto += case_3; // anyInt == 3

        } );

        // здесь будет метка выхода с таблицы
        wait( 2500 );
        end_thread();
    }

    private void case_0( LabelCase label ) {
        Comment = "case #0";
        jump( label.EndJumpTable ); // переход на метку выхода вне конструкции "jump_table"
    }

    private void case_1( LabelCase label ) {
        Comment = "case #1";
        jump( label.EndJumpTable );
    }

    private void case_2( LabelCase label ) {
        Comment = "case #2";
        jump( label.EndJumpTable );
    }

    private void case_3( LabelCase label ) {
        Comment = "case #3";
        jump( label.EndJumpTable );
    }

}
```
