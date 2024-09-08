# Стартер

Этот плагин позволит быстро сделать простой стартер миссий. Его можно создавать много раз. Код стартер имеет такой вид:

```csharp
public partial class MAIN : Thread {

    // здесь будем хранить ссылку на стартер
    public static Starter STARTER_EXAMPLE;

    public override void START( LabelJump label ) {
        create_thread<STARTER>();
    }

    //----------------------------------------------------------------------------------------------------

    public class STARTER : Thread {

        public override void START( LabelJump label ) {

            // setup - объект специального класса, который позволит настроить стартер
            STARTER_EXAMPLE = new Starter( setup => {
                // ...
            } );

        }

    }

}
```

Объект **STARTER\_EXAMPLE** мы может использовать в миссиях, чтобы увеличить счётчик или создать поток **STARTER** ещё раз. Это нужно делать уже в миссиях:

```csharp
public partial class MAIN : Thread {

    //----------------------------------------------------------------------------------------------------

    public class MISS1 : Mission {

        public override void START( LabelJump label ) {
            wait( DefaultWaitTime );
            fade( FadeType.IN, DefaultWaitTime );
            PlayerChar.can_move( true );
            PlayerActor.set_immunities( 0, 0, 0, 0, 0 );
            wait( 10000 );
            jump_passed();

            OnPassed = delegate {
                // запустить скрипт стартера и увеличить счётчик на 1
                STARTER_EXAMPLE.recreate( true );
            };

            OnFailed = delegate {
                // запустить скрипт стартера без изменения счётчика
                STARTER_EXAMPLE.recreate();
            };

        }

    }

    public class MISS2 : Mission {

        public override void START( LabelJump label ) {
            wait( DefaultWaitTime );
            fade( FadeType.IN, DefaultWaitTime );
            PlayerChar.can_move( true );
            PlayerActor.set_immunities( 0, 0, 0, 0, 0 );
            wait( 10000 );
            jump_passed();

            OnPassed = delegate {
                // запустить скрипт стартера и увеличить счётчик на 1
                STARTER_EXAMPLE.recreate( true );
            };

            OnFailed = delegate {
                // запустить скрипт стартера без изменения счётчика
                STARTER_EXAMPLE.recreate();
            };

        }

    }

    public class MISS3 : Mission {

        public override void START( LabelJump label ) {
            wait( DefaultWaitTime );
            fade( FadeType.IN, DefaultWaitTime );
            PlayerChar.can_move( true );
            PlayerActor.set_immunities( 0, 0, 0, 0, 0 );
            wait( 10000 );
            jump_passed();

            OnPassed = delegate {
                // запустить скрипт стартера и увеличить счётчик на 1
                STARTER_EXAMPLE.recreate( true );
            };

            OnFailed = delegate {
                // запустить скрипт стартера без изменения счётчика
                STARTER_EXAMPLE.recreate();
            };

        }

    }
	
}
```

Мы должны указать стартеру какие миссии вызывать, какое название выводить на экране и какие условия для этого нужны. Объект **setup** как раз поможет это сделать:

```csharp
public partial class MAIN : Thread {

    // здесь будем хранить ссылку на стартер
    public static Starter STARTER_EXAMPLE;

    public override void START( LabelJump label ) { 
        create_thread<TEST>();
    }

    //----------------------------------------------------------------------------------------------------

    public class TEST : Thread {

        public override void START( LabelJump label ) {

            STARTER_EXAMPLE = new Starter( setup => {

                // изменим иконку радара, стандартно установлено "CJ"
                setup.set_radar_icon( RadarIconID.RYDER );

                // изменим место старта, стандартно установлено 0.0 0.0 0.0
                setup.set_start_position( 2488.562, -1666.865, 12.8757 );

                // изменим радиус действия, стандартно установлено 1.0
                setup.set_radius( 1.5 );

                // регулирует длительность затемнения, стандартно установлено 250
                //h.set_fade_time( 1500 );

                // добавим миссию без условий запуска и GTX-названия миссии
                setup.add_mission<MISS1>();

                // добавим миссию с условем запуска
                setup.add_mission<MISS2>( "GXT_MS2", PlayerActor.is_in_vehicle_with_model( CarModel.CHEETAH ) );

                // добавим ещё одну миссию
                setup.add_mission<MISS3>( "GXT_MS3" );

            } );

        }

    }

}
```

В стартере уже установлены базовые условия запуска, а миссия "MIS2" будет требовать ещё одно условие.

{% hint style="danger" %}
Плагин должен создаваться в отдельных потоках (не **MAIN**)!
{% endhint %}

{% hint style="warning" %}
Мы можем использовать плагин много раз, но при условии, что он единственный в потоке!
{% endhint %}
