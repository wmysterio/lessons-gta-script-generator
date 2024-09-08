# Группировка миссий

Если в проекте планируется набор из небольших миссий, то есть смысл сделать группировку миссий (объединить небольшие миссии в одну большую). В этом случае мы можем воспользоваться [таблицей переходов](../base/jump\_tables.md), чтобы разделить миссию не отдельные части.

В качестве переменной мы можем использовать счётчик выполненных миссий или использовать отдельную переменную, отвечающею за этап (часть) миссии.

```csharp
partial class MAIN {

    static Int CJ_MISSION_PASSED;

    public class MYMISS : Mission {
    
        public override void START( LabelJump label ) {
            OnPassed = DEFAULT_PASSED;
            OnFailed = DEFAULT_FAILED;
            OnClear = DEFAULT_CLEAR;
            //----------------------------------------
            
            wait( 1000 );
            //...
            jump_table( CJ_MISSION_PASSED, table => {
                table.Auto += PART_0; // CJ_MISSION_PASSED == 0
                table.Auto += PART_1; // CJ_MISSION_PASSED == 1
                table.Auto += PART_2; // CJ_MISSION_PASSED == 2
                table.Auto += PART_3; // CJ_MISSION_PASSED == 3
                table.Auto += PART_4; // CJ_MISSION_PASSED == 4
            } );
            @return(); // if not match
        }

        #region SUBMISSION
        private void PART_0( LabelCase l ) {
            jump_passed();
        }
        private void PART_1( LabelCase l ) {
            jump_passed();
        }        
        private void PART_2( LabelCase l ) {
            jump_passed();
        }
        private void PART_3( LabelCase l ) {
            jump_passed();
        }
        private void PART_4( LabelCase l ) {
            jump_passed();
        }
        #endregion
        
        //----------------------------------------
            
        private void DEFAULT_PASSED() {
            show_text_styled( sString.M_PASSD, 5000, 1 );
            play_music( MusicID.MISSION_PASSED );
            CJ_MISSION_PASSED += 1;
            // ...
        }
        
        private void DEFAULT_FAILED() {
            show_text_styled( sString.M_FAIL, 5000, 1 );
            // ...
        }
        
        private void DEFAULT_CLEAR() {
            // ...
        }
        
    }

}
```

Теперь в зависимости от значения переменной `CJ_MISSION_PASSED` будет выполнятся только одна конкретная часть миссии (`PART_X`).

Такой подход требует компактной реализации награды для игрока. Есть две базовые награды за прохождение миссии: уважение и деньги. Иногда они могут быть вместе, иногда награда отсутствует. В этом случае мы можем использовать три переменные, чтобы распределить награду в зависимости от этапа миссии.

```csharp
public class MYMISS : Mission {

    Int plus_reward, plus_respect, plus_temp;

    //...
    
    private void DEFAULT_PASSED() {
        //show_text_styled( sString.M_PASSD, 5000, 1 );
        //play_music( MusicID.MISSION_PASSED );
        and( plus_reward > 0, delegate { plus_temp += 1; } );
        and( plus_respect > 0, delegate { plus_temp += 2; } );
        and( plus_temp == 3, delegate { 
            show_text_1number_styled( sString.M_PASSS, plus_reward, 5000, 1 );
            add_respect( plus_respect );
            PlayerChar.add_money( plus_reward ); 
        } );
        and( plus_temp == 2, delegate { 
            show_text_styled( sString.M_PASSR, 5000, 1 );
            add_respect( plus_respect ); 
        } );
        and( plus_temp == 1, delegate { 
            show_text_1number_styled( sString.M_PASS, plus_reward, 5000, 1 );
            PlayerChar.add_money( plus_reward ); 
        } );
        and( plus_temp == 0, delegate { 
            show_text_styled( sString.M_PASSD, 5000, 1 ); 
        } );
        and( plus_temp > -1, delegate { 
            play_music( MusicID.MISSION_PASSED );
        } );
        CJ_MISSION_PASSED += 1;
    }
    
}
```

Если всё оставить как есть, то при прохождении этапов будет показано сообщение о выполненной миссии. Если этап имеет награду за прохождение, то достаточно указать переменным `plus_reward` и `plus_respect` новые значения:

```csharp
public class MYMISS : Mission {

    //...
    
    #region SUBMISSION
    private void PART_0( LabelCase l ) {
        plus_temp.value = -1;
        jump_passed();
    }
    
    private void PART_1( LabelCase l ) {
        jump_passed();
    }   
         
    private void PART_2( LabelCase l ) {
        plus_respect.value = 5; 
        jump_passed();
    }
    
    private void PART_3( LabelCase l ) {
        plus_reward.value = 5000;
        jump_passed();
    }
    
    private void PART_4( LabelCase l ) {
        plus_respect.value = 15; 
        plus_reward.value = 25000;
        jump_passed();
    }
    #endregion
    
}
```

Теперь при прохождении этапа `PART_0` никаких сообщений не будет и награды тоже (в том числе и звука). За прохождение `PART_1` будет показано сообщение о выполненной миссии. За `PART_2` будет повышено уважение, за `PART_3` дадут 5000$; а за последний этап будет повышено уважение и дадут 25000$ одновременно.

С успешным выполнением миссии мы разобрались. Давайте теперь рассмотрим код провала. У нас могут быть различные ситуации, которые приводят к провалу. Чаще всего выводится текст с информацией о причине неудачи. Мы можем использовать переменную, которая будет хранить GXT-ключ и написать метод, который будет задавать этой переменной текст провала и с последующим прыжком на метку провала.

```csharp
public class MYMISS : Mission {

    // ...
    
    sString failedMessage;
    
    void ___jump_failed_message( string gxt ) {
        failedMessage.value = gxt;
        jump_failed();
    }
    
    //----------------------------------------
        
    public override void START( LabelJump label ) {
        failedMessage.value = sString.DUMMY;
        //...
    }
    
    #region SUBMISSION
    private void PART_0( LabelCase l ) {
        plus_temp.value = -1;
        LocalTimer1.value = 0;
        Cycle += P0_LOOP; // without jump!
    }
    // ...
    #endregion
    
    private void P0_LOOP {
        wait( 0 );
        and( PlayerActor.is_in_any_vehicle(), delegate {
            ___jump_failed_message( "GXTNAME" );
        } );
        and( LocalTimer1 > 10000, delegate {
            jump_passed();
        } );
    }
    
    private void DEFAULT_FAILED() {
        show_text_styled( sString.M_FAIL, 5000, 1 );
        and( failedMessage != sString.DUMMY, delegate { 
            show_text_lowpriority( failedMessage, 6000, 1 ); 
        } );
        // ...
    }
    
}
```

Теперь если игрок будет в любой машине, то миссия будет провалена. При этом будет показано сообщение из GXT-ключа `GXTNAME`. Если игрок не сядет в транспорт в течении 10 секунд, то миссия будет выполнена.

Нам осталось разобраться только с очисткой миссии. В этом блоке идёт не только удаление всех созданных субъектов, но и возврат всех настроек в изначальное состояние. Игра часто самостоятельно делает этот возврат, но лучше принудительно это делать (для страховки).

Ситуацию усложняет то, что в этом примере сразу несколько миссий. Поэтому мы должны делать очистку с учётом всех изменений:

```csharp
public class MYMISS : Mission {

    // ...

    #region SUBMISSION
    private void PART_0( LabelCase l ) {
        set_vehicle_traffic_density_multiplier( 0.0 );
        jump_passed();
    }
    private void PART_1( LabelCase l ) {
        set_ped_traffic_density_multiplier( 0.0 );
        jump_passed();
    }        
    //...
    #endregion
    
    //----------------------------------------
        
    private void DEFAULT_CLEAR() {
        set_vehicle_traffic_density_multiplier( 1.0 );
        set_ped_traffic_density_multiplier( 1.0 );
    }
    
}
```

На этапе 0 у нас отключается трафик пешеходов, а на этапе 1 — трафик транспорта. Однако в методе `DEFAULT_CLEAR` мы должны сделать возврат к стандартным настройкам оба параметра независимо от этапа. Этот принцип должен соблюдаться всегда, чтобы гарантировать полное соответствие настроек до миссии и после неё.
