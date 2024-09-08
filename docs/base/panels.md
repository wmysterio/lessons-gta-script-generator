# Панели

В GTA SA мы можем скриптом создавать меню с выбором элемента. Эта возможность присутствует и в генераторе. Я решил написать о ней более подробно, так как здесь есть несколько моментов, которые могут быть полезными.

Сами методы уже давно знакомы скриптерам. Генератор расширяет их для упрощения написания кода. Прежде всего теперь не надо указывать пустые строки (**'DUMMY'**) для рядов колонки. Это делается автоматически:

```csharp
public class TEST : Thread {

    Panel panel;

    public override void START( LabelJump label ) {

        panel.create( "HEADER", 10.0, 10.0, 180.0, 4, 1, 1, PanelAlign.LEFT );
        panel.set_column_data( 0, "COLUMN0", "ROW0" );
        panel.set_column_data( 1, "COLUMN1", "ROW1" );
        panel.set_column_data( 2, "COLUMN2", "ROW2" );
        panel.set_column_data( 3, "COLUMN3", "ROW3" );

        end_thread();
    }

}
```

Это будет иметь "классический" страшный вид:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
08D4: 0@ = create_panel_with_title 'HEADER' position 10.0 10.0 width 180.0 columns 4 interactive 1 background 1 alignment 1
08DB: set_panel 0@ column 0 header 'COLUMN0' data 'ROW0' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 1 header 'COLUMN1' data 'ROW1' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 2 header 'COLUMN2' data 'ROW2' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 3 header 'COLUMN3' data 'ROW3' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
004E: end_thread
```

Таким образом ми сможем убрать лишнюю рутину. Также этот метод может принимать массив в качестве данных, что уже очень упрощает некоторые задачи:

```csharp
public class TEST : Thread {

    Panel panel;
    Array<sString> rowData = 4;

    public override void START( LabelJump label ) {

        panel.create( "HEADER", 10.0, 10.0, 180.0, 1, 1, 1, PanelAlign.LEFT );

        rowData[ 0 ].value = "ROW0";
        rowData[ 1 ].value = "ROW1";
        rowData[ 2 ].value = "ROW2";
        rowData[ 3 ].value = "ROW3";

        panel.set_column_data( 0, "COLUMN0", rowData ); // передаём массив качестве аргумента
        end_thread();
    }

}
```

Размер массива не должен превышать 12 элементов. Если их меньше этого количества, то оставшиеся места для панели будут заполнены пустыми строками:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
08D4: 0@ = create_panel_with_title 'HEADER' position 10.0 10.0 width 180.0 columns 1 interactive 1 background 1 alignment 1
05AA: 1@s = 'ROW0' // @s = ? (sString)
05AA: 3@s = 'ROW1' // @s = ? (sString)
05AA: 5@s = 'ROW2' // @s = ? (sString)
05AA: 7@s = 'ROW3' // @s = ? (sString)
08DB: set_panel 0@ column 0 header 'COLUMN0' data 1@s 3@s 5@s 7@s 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
004E: end_thread
```

Есть ещё одна возможность создавать панели. Метод `create` имеет дополнительную перегрузку. Он принимает многомерный массив с описанием всех строк и колонок:

```csharp
public class TEST : Thread {

    Panel panel;

    public override void START( LabelJump label ) {

        var panelData = new string[,] {
            //-------------------------------------------------------------
            {     "COLUMN0",     "COLUMN1",     "COLUMN2",     "COLUMN3"  },    // сначала указываем заголовки колонок
            //-------------------------------------------------------------
            {     "ROW0",        "ROW0",        "ROW0",        "ROW0"     },    // строка #0
            {     "ROW1",        "ROW1",        "ROW1",        "ROW1"     },    // строка #1
            {     "ROW2",        "ROW2",        "ROW2",        "ROW2"     },    // строка #2
            {     "ROW3",        "ROW3",        "ROW3",        "ROW3"     }     // строка #3
            //-------------------------------------------------------------
        };

        panel.create( "PANEL", 10.0, 10.0, 180.0, 1, 1, PanelAlign.LEFT, panelData );

        end_thread();
    }

}
```

Я сделал расположение элементов массива так, как это будет выглядеть в игре. Количество колонок определяет метод, поэтому его писать в качестве аргумента не надо. Вот такой код будет в итоге:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
08D4: 0@ = create_panel_with_title 'PANEL' position 10.0 10.0 width 180.0 columns 4 interactive 1 background 1 alignment 1
08DB: set_panel 0@ column 0 header 'COLUMN0' data 'ROW0' 'ROW1' 'ROW2' 'ROW3' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 1 header 'COLUMN1' data 'ROW0' 'ROW1' 'ROW2' 'ROW3' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 2 header 'COLUMN2' data 'ROW0' 'ROW1' 'ROW2' 'ROW3' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
08DB: set_panel 0@ column 3 header 'COLUMN3' data 'ROW0' 'ROW1' 'ROW2' 'ROW3' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 'DUMMY' 
004E: end_thread
```

Этот способ отлично подойдёт для тех случаев, когда у нас используется только одна панель без зависимых переменных.
