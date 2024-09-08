# Продвинутый стартер

В отличии от плагина [Starter](plugin\_starter.md), этот имеет больше гибкости. Установка позиции и иконки можно осуществлять в любом потоке, миссии и внешнем скрипте. Также добавлена специальная переменная, которая отвечает за этап миссии. Её стоит использовать в том случае, когда одна миссия имеет несколько своих под-миссий (чтобы уменьшить общее количество миссий). Ниже подан пример реализации такого стартера:

```csharp
using GTA;
using GTA.Plugins;

public partial class MAIN : Thread {

    // здесь будем хранить ссылку на стартер
    public static StarterAdvanced STARTER_EXAMPLE;

    public override void START( LabelJump label ) { 
        create_thread<TEST>();
    }

    //----------------------------------------------------------------------------------------------------

    public class TEST : Thread {

        public override void START( LabelJump label ) {

            STARTER_EXAMPLE = new StarterAdvanced( setup => {
                setup.set_radar_icon( RadarIconID.RYDER );
                setup.set_start_position( 2488.562, -1666.865, 12.8757 );
                setup.add_mission<MISS1>();
                setup.add_mission<MISS2>( "GXT_MS2", PlayerActor.is_in_vehicle_with_model( CarModel.CHEETAH ) );
                setup.add_mission<MISS3>( "GXT_MS3" );
            } );

        }

    }

}
```

Код миссий мы можем (для примера) написать вот так:

```csharp
public partial class MAIN : Thread {

    public class MISS1 : Mission {

        public override void START( LabelJump label ) {

            OnPassed = delegate { STARTER_EXAMPLE.recreate( true ); };
            OnFailed = delegate { STARTER_EXAMPLE.recreate(); };
    
        }

    }

    public class MISS2 : Mission {

        public override void START( LabelJump label ) {

            OnPassed = delegate {
                // изменяем иконку и позицию перед запуском стартера
                STARTER_EXAMPLE.change_icon( RadarIconID.BIG_SMOKE );
                STARTER_EXAMPLE.change_position( 1440.0, 144.0, 43.0 );
                STARTER_EXAMPLE.recreate( true );
            };
            OnFailed = delegate { STARTER_EXAMPLE.recreate(); };

        }

    }

    public class MISS3 : Mission {

        public override void START( LabelJump label ) {

            OnPassed = delegate {
                // задаём название сейву из GXT-названия миссии
                set_latest_mission_passed( STARTER_EXAMPLE.CurrentGXT );

                // не увеличиваем счётчик, а увеличиваем этап
                STARTER_EXAMPLE.recreate( false, true );
            };
            OnFailed = delegate { STARTER_EXAMPLE.recreate(); };
        }

    }

}
```

После прохождения миссии **MISS2** стартер будет создаваться в новых координатах и иметь другую иконку. Использование расширенного стартера требует определённой практики. Если есть сомнения, то можете пользоваться обычным стартером или создать собственный.

{% hint style="info" %}
Требования к плагину такие же, как и в плагине [Starter](plugin\_starter.md)!
{% endhint %}
