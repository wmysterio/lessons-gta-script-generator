# FXT тексты

Генератор скриптов поддерживает экспорт текстов в FXT архив. Начиная с версии 7.0 есть возможность экспорта сразу в несколько файлов. По умолчанию все тексты будут экспортироваться в файл **text.fxt** без кодировки.

### Добавление текста и конвертация

Для добавления записей существует свойство `FXT`. Метод `add` добавляет новую запись в текущий файл:

```csharp
public class TEST : Thread {

    public override void START( LabelJump label ) {


        FXT.add( "KEY_002", "This is my text!" );


        show_text_highpriority( "KEY_002", 5000, 0 );
        end_thread();
    }

}
```

Метод принимает название GXT-ключа в первый параметр, а во втором параметре указываем сам текст. В этом случае GXT-ключ будет сохранён, что не позволит в текущий файл добавить ещё одну запись с таким же ключом. Сам ключ используется в командах скрипта (например, в **show\_text\_highpriority**).

### Работа с кодировками

В предыдущем примере мы рассмотрели добавление записей, но они никак не кодируются и записываются как есть. Кириллица в этом случае не будет правильно отображаться в игре (как и многие другие языки, которые не предусмотрены разработчиками). В этом случае нам поможет система кодировок.

Суть кодировок состоит в том, что на месте "правильного" символа в текстах локализации пишется другой, который поддерживается оригинальным набором символов. Каждый локализатор кодирует символы по своему, поэтому их надо знать заранее. Для режима GTA SA генератор скриптов имеет одну встроенную кодировку: **SmartLoc**. Для её установки надо воспользоваться методом:

```csharp
using GTA.Core; // <-- класс "GXTEncoding" для работы с кодировками находится в этом пространстве имён

public class TEST : Thread {

    public override void START( LabelJump label ) {


        FXT.set_encoding( GXTEncoding.RU_SmartLoc ); // <-- устанавливаем текущему файлу новую кодировку


        FXT.add( "KEY_002", "Теперь игра будет правильно кириллицу!" ); // ЏeЈep© њ™pa —yљe¦ Јpaўњћ©®o kњpњћћњ y
        show_text_highpriority( "KEY_002", 5000, 0 );
        end_thread();
    }

}
```

Если планируется использовать персональную кодировку, то для этого используется специальный класс — `GXTEncoding`. Пример установки показан ниже:

```csharp
using GTA.Core; // <-- класс "GXTEncoding" для работы с кодировками находится в этом пространстве имён

public class TEST : Thread {

    public override void START( LabelJump label ) {
	
        // все символы с массива "normal" будут преобразованы в символы "symbol"
        char[] normal = new char[] { 'Н', 'Г', 'С' };
        char[] symbol = new char[] { 'N', 'G', 'S' };

        GXTEncoding newEnc = new GXTEncoding( normal, symbol, false ); // <-- создание новой кодировки
        FXT.set_encoding( newEnc ); // <-- установка персональной кодировки


        FXT.add( "KEY_002", "НГГССНС!" ); // NGGSSGS
        show_text_highpriority( "KEY_002", 5000, 0 );
        end_thread();
    }

}
```

Конструктор класса **GXTEncoding** принимает 3 параметра, но последний является опциональным. Если передать значение **true**, то записи будут конвертированы в верхний регистр, что может упростить жизнь с некоторыми кодировками, которые не используют строчные буквы.

### Работа с файлами

До этого момента мы работали только с одним файлом. В большинстве случаев этого достаточно, но иногда бывают ситуации, когда надо создать тексты для разных языков или для разных кодировок. Для таким случаем есть метод, который добавляет новый файл:

```csharp
FXT.set_file( "fileName" );
```

Все записи, добавленные в коде после метода, будут добавлены в указанный файл. Обратите внимание, что установка нового файла сбрасывает кодировку, поэтому её нужно указать ещё раз. Давайте рассмотрим пример, когда используется 2 файла:

```csharp
public class TEST : Thread {

    public override void START( LabelJump label ) {
	
        FXT.set_file( "english" )
           //.set_encoding( GXTEncoding.None ) // <-- для английской версии кодировка не обязательна
           .add( "DIALOG1", "Hello!" )
           .add( "DIALOG2", "I am CJ!" )
           .add( "DIALOG3", "Phone ring!" )
           .add( "DIALOG4", "Later!" );

        FXT.set_file( "russian" )
           .set_encoding( GXTEncoding.RU_SmartLoc )
           .add( "DIALOG1", "Привет!" )
           .add( "DIALOG2", "Я CJ!" )
           .add( "DIALOG3", "Звук телефона!" )
           .add( "DIALOG4", "Пока!" );

        show_text_highpriority( "DIALOG1", 5000, 0 );
        end_thread();
    }

}
```

Оба файла будут экспортированы в указанную директорию и Вам останется только предложить конечному пользователю выбрать один из них. Если создавать файлы на одном языке, но с разными кодировками, то нет необходимости дублировать код. Мы можем скопировать все записи, используя метод `copy`:

```csharp
public class TEST : Thread {

    public override void START( LabelJump label ) {
	
        char[] normal = new char[] { 'И', 'Ж', 'С' /* ... */ };
        char[] symbol = new char[] { 'N', 'G', 'C' /* ... */ };

        GXTEncoding encodingFrom1C = new GXTEncoding( normal, symbol, true );
		
        FXT.set_file( "SmartLocFile" )
           .set_encoding( GXTEncoding.RU_SmartLoc )
           .add( "DIALOG1", "Привет!" )
           .add( "DIALOG2", "Я CJ!" )
           .add( "DIALOG3", "Звук телефона!" )
           .add( "DIALOG4", "Пока!" );

        FXT.set_file( "1СFile" )
           .set_encoding( encodingFrom1C )
           .copy( "SmartLocFile" ); // <-- копирует GXT-ключи и записи с файла "SmartLocFile"

        show_text_highpriority( "DIALOG1", 5000, 0 );
        end_thread();
    }

}
```

{% hint style="warning" %}
Файлы без записей экспортироваться не будут!
{% endhint %}

{% hint style="danger" %}
Созданные ранее файлы (имена которых не задействованы в скрипте) не будут удалены, как было в старых версиях!
{% endhint %}

### Разное

Для удобства лучше создать отдельный класс, где будут храниться тексты. Это поможет при разработке, так как их не надо будет искать по всему проекту. Пример такого класса:

```csharp
using static GTA.Core.Script; // <-- нужно для простого доступа к свойству "FXT"

public static class MyTexts {
    public static void Init() {

        FXT.set_encoding( GXTEncoding.RU_SmartLoc ) 
           .add( "DIALOG1", "Привет!" )
           .add( "DIALOG2", "Я CJ!" )
           .add( "DIALOG3", "Звук телефона!" )
           .add( "DIALOG4", "Пока!" );
           // тут другие записи...

    }
}

// ...

public partial class MAIN : Thread {

    public override void START( LabelJump label ) {
        MyTexts.Init(); // <-- создаём тексты только 1 раз

        // ...
        create_thread<TEST>();
        // ...

    }

}

// ...

public partial class TEST : Thread {

    public override void START( LabelJump label ) {
        show_text_highpriority( "DIALOG1", 5000, 0 ); // используем
        wait( 5000 );
        show_text_highpriority( "DIALOG2", 5000, 0 ); // используем
        wait( 5000 );
        show_text_highpriority( "DIALOG3", 5000, 0 ); // используем
        wait( 5000 );
        end_thread();
    }

}
```
