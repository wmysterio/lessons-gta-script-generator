# Телефон

Начиная с версии 5.1.5 можно использовать плагин `Phone`. Он реализует систему приёма звонков, как это было в оригинальной игре. Создаётся он так:

```csharp
using GTA;
using GTA.Plugins;

public partial class MAIN { 

    static Phone CJ_PHONE; // сохраняем ссылку на плагин

    public override void START( LabelJump label ) {
        create_thread<PHONE>();
        end_thread();
    }
	
    //----------------------------------------------------------------------------------------------------

    public class PHONE : Thread {
        
        public override void START( LabelJump label ) {
        
            CJ_PHONE = new Phone( setup => {

                // можно указать время, которое нужно ождать перед звонком
                // setup.set_max_time_between_calls( 60000 );

                // максимальное время на ответ игрока
                // setup.set_max_response_time( 20000 );

                // можно указать номер WAV-файла, если стандартная мелодия не устраивает
                // setup.set_ringtone( 23000 );

                setup.add_dialog( 11, dialog => {

                    dialog.add_replica( "DIALOG1", 4000, false );
                    dialog.add_replica( "DIALOG2", 6000, true );
                    dialog.add_replica( "DIALOG3", 3500, false );
                    dialog.add_replica( "DIALOG4", 4000, true );
                    dialog.add_replica( "DIALOG5", 3500, false );

                    dialog.OnComplete = delegate {
                        create_thread<TEMP>();
                    };

                } );

            } );

        }

    }

    public class TEMP : Thread {

        public override void START( LabelJump label ) {
            end_thread();
        }

    }

}
```

Объект **setup** позволяет настроить телефон. Обязательным методом является `add_dialog`. Он создаёт новый диалог. Здесь **11** - идентификатор диалога. Все остальные методы опциональны.

{% hint style="danger" %}
Все диалоги должны иметь разные ID!
{% endhint %}

В объекте **setup** доступны новые свойства: `OnLoadData`, `OnUnloadData` и `OnReplicaChanged`. Первое свойство позволяет указать метод, который будет выполнятся при загрузке данных диалога. Второе свойство позволяет указать действие когда загруженные данные освобождаются. Последнее свойство позволяет вызывать указанный метод, когда выполняется каждая реплика:

```csharp
public class PHONE : Thread {
    
    public override void START( LabelJump label ) {
    
        CJ_PHONE = new Phone( setup => {

            setup.OnLoadData = d => {
                // d.DialogID
                // d.TotalReplicas
            };
            setup.OnUnloadData = delegate {
                // ...
            };
            setup.OnReplicaChanged = delegate { 
                // ...
            };

            // код добавления диалогов
            
        } );

    }
    
}
```

Объект **dialog** позволяет настроить диалог. В нём есть основной метод `add_replica`. Он принимает 3 параметра: GXT-ключ строки, которая будет показана; длительность реплики; далее значение **true**, если надо, чтобы игрок шевелил губами во время этой реплики или **false**, чтобы НЕ шевелил.

Реплики будут выполнятся последовательно. После последней фразы игрок спрячет телефон и диалог закончится. Свойство `OnComplete` позволяет выполнить действие после успешного завершения диалога. В данном примере создаётся поток **TEMP**. Осталось только научится делать звонок:

```csharp
using GTA;
using GTA.Plugins;

public partial class MAIN : Thread {

    static Phone CJ_PHONE;

    public override void START( LabelJump label ) {
        create_thread<PHONE>();
        wait( 1000 ); // лучше сделать небольшую задержку после создания потока "PHONE".
        // ...
		
		
        CJ_PHONE.call( 11 ); // <-- активируем телефон, передав в метод идентификатор диалога
		
		
        // ...
        end_thread();
    }

    // здесь класс "MAIN.PHONE" ...

}
```

{% hint style="danger" %}
Плагин может существовать только в единственном экземпляре и требует отдельный поток (не **MAIN**)!
{% endhint %}

{% hint style="warning" %}
Использование плагина требует хорошего планирования цепочек миссий, чтобы звонки совершались в строгой последователоьности согласно плану. В противном случае есть риск, что ID звонка может перезаписаться, что не позволит выполнить диалог и действия после его завершения!
{% endhint %}
