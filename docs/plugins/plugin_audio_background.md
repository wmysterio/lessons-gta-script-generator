# Аудио фон

Плагин используется для воспроизведения одного аудио-файла в качестве фоновой музыки. Его можно использовать в миссиях, где должно быть звуковое сопровождение. За это отвечает класс **AudioBackground**. Давайте рассмотрим пример использования:

```csharp
using GTA;
using GTA.Plugins;

public partial class MAIN : Thread {

    // здесь будем хранить ссылку на аудио-фон
    public static AudioBackground AUDIO_BACKGROUND;

    public override void START( LabelJump label ) {

        // ...
        create_thread<AUDIOBG>(); // запускаем поток с инициализацией фона
        create_thread<TEST>(); // использовать плагин нужно только после инициализации!
        end_thread();
    }

    //----------------------------------------------------------------------------------------------------

    public class AUDIOBG : Thread {

        public override void START( LabelJump label ) {
            AUDIO_BACKGROUND = new AudioBackground();
        }

    }

    //----------------------------------------------------------------------------------------------------

    public class TEST : Thread {

        public override void START( LabelJump label ) {

            // загружаем звук с именем 0 (расширение указывать не нужно, оно по умолчанию "*.mp3")
            // в данном случае мы ищем файл "0.mp3"
            // false/true (поумолчанию false): зацикливания звука (опциональный параметр)
            AUDIO_BACKGROUND.load( 0, false );

            wait( AUDIO_BACKGROUND.is_ready ); // ожидаем загрузки звука

            AUDIO_BACKGROUND.play(); // воспроизводим аудио-файл
            wait( 10000 ); // ожидаем 10 секунд

            AUDIO_BACKGROUND.unload(); // выгружаем аудио-файл

            wait( AUDIO_BACKGROUND.is_stopped ); // ожидаем завершения работы

            end_thread();
        }

    }

}
```

Поиск нужного файла по умолчанию осуществляется в папке с игрой (где **gta\_sa.exe**). Если нужно указать другую папку, то используем метод `chdir`:

```csharp
chdir( "CLEO" );
AUDIO_BACKGROUND.load( 0 ); // <-- "ПАПКА_С_ИГРОЙ\CLEO\0.mp3"
```

Метод **chdir** нужно применять перед методом `load` каждый раз! Если файл не удалось найти или загрузить, то звук играть не будет. И последнее с важного — начиная с версии 7.1 есть возможность регулировать громкость трека:

```csharp
AUDIO_BACKGROUND.set_volume( 0.2 ).play(); // изменяем до начала воспроизведения
wait( 10000 );
AUDIO_BACKGROUND.set_volume( 1.2 ); // изменяем спустя какое-то время
```

{% hint style="warning" %}
&#x20;Метод **set\_volume** нужно применять после метода **load** и до метода **unload**!
{% endhint %}

{% hint style="warning" %}
Плагин использует файл с расширением **mp3**. А название должно быть только числовыми!
{% endhint %}

{% hint style="danger" %}
Плагин может создаваться только один раз и в отдельном потоке (не **MAIN**)! Плагин рекомендуется использовать только в одном активном потоке!
{% endhint %}

{% hint style="danger" %}
Если в одном потоке использовать плагин вместе с плагином [Аудио-плеер](plugin\_audio\_player.md), то делать загрузку нужно после проверки **is\_ready** или до метода **load** с проверкой **is\_ready**.
{% endhint %}
