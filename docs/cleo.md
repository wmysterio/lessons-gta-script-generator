# CLEO

Версия 7.5 дарит возможность генерировать CLEO-скрипты! Принцип написания скриптов и основные возможности не отличаются от основной ветки генератора. Но в этом режиме есть некоторые особенности, о которых стоит рассказать.

Подключение и настройка написана в разделе [Установка и настройка. Компиляция скриптов](../settings.md#kompilyaciya-skriptov). Главное отличие в этой настройке: нам надо подключать библиотеки `GTA.III.CLEO`, `GTA.VC.CLEO` или `GTA.SA.CLEO`. Также здесь нужно указывать папку с игрой, так как метод **SetMainSCMFolder** недоступен:

```csharp
namespace demo_app {

    class Program {

        static void Main( string[] args ) {

            // Куда сохранять FXT-файлы
            Generator.SetFXTFolder( @"D:\Programm\GTA_SA Career v2.0\modloader\wmysterio\cleo\cleo_text" );

            // Путь к игре
            Generator.SetGTAFolder( @"D:\Programm\GTA_SA Career v2.0" );
            
            // Путь к "sanny.exe"
            Generator.SetSannyBuidlerFolder( @"D:\Programm\Sanny Builder 3" );

            //Generator.Start<MAIN>();
            Generator.Compile<MAIN>( false );

            Console.ReadKey();

        }

    }

}
```

Между символами `<` и `>` указывается тип, который наследует `Thread`, `Mission` или `MissionCustom`. Чтобы создать CLEO-поток, достаточно этого кода:

```csharp
public class MAIN : Thread {

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Мы видим, что никак отличий в коде в сравнении с MAIN.scm потоком не наблюдается. Команда `end_thread` будет заменена на опкод `0A93: end_custom_thread`. Такие же манипуляции будут с другими командами, которые используются только в CLEO.

Обратите внимание на название класса. Генератор будет использовать его как название файла. Особых ограничений на название не накладывается, но рекомендую использовать прописные символы в имени класса.

По умолчанию скрипт компилируется с расширением **CS** или **СМ** в папку `ИГРА\CLEO`. Если нужно изменить эти свойства, то здесь используется система атрибутов:

```csharp
[Path(@"CLEO\NEW_DIR")]
[Extension( "H" )]
[EnableThreadSaving]
public class MAIN : Thread {

    public override void START( LabelJump label ) {
        end_thread();
    }

}
```

Всего таких атрибутов 3; Вы можете использовать только те, которые нужны, игнорируя остальные. Код выше сохранит скрипт по этому пути: `ИГРА\CLEO\NEW_DIR\MAIN.H`.

{% hint style="danger" %}
Убедитесь, что путь содержит все нужные папки на жестком диске!
{% endhint %}

Атрибут `EnableThreadSaving` реализует опкод `0A95`, но без его явного вызова. Этот атрибут, как и `Extension`, не применяется к миссиям и будет игнорироваться.

### Компиляция нескольких скриптов

Если есть необходимость компиляции целой пачки скриптов, то нам достаточно несколько раз вызывать метод **Compile**:

```csharp
Generator.Compile<SCRIPT1>();
Generator.Compile<SCRIPT2>();
Generator.Compile<SCRIPT3>();
//...
Generator.Compile<SCRIPTN>();
```

Обратите внимание, что если использовать команду

```csharp
create_thread<SCRIPT_NAME>();
```

то поток **SCRIPT\_NAME** будет включен в финальную сборку и скомпилируется самостоятельно вместе с исходным скриптом!
