# Внешние скрипты

Генератор поддерживает внешние скрипты в библиотеке **GTA.SA**. Все внешние скрипты должны наследовать класс `External`:

```csharp
public class EXT : External {

    public override void START( LabelJump label ) {

        and( !PlayerActor.is_dead(), delegate {
            PlayerActor.task.die();
        } );

        end_thread();
    }

}
```

Сам скрипт не будет работать. Сначала нам нужно загрузить скрипт и запустить его. Вот так это можно сделать:

```csharp
public class TEST : Thread {

    static Int ScriptStatus;

    public override void START( LabelJump label ) {
    
        //...
        
        Jump += LOOP;
    }
    
    private void LOOP( LabelJump label ) {
        wait( 0 );            
        Script.and( PlayerChar.is_defined(), delegate {
            
            Script.get_external_script_status<EXT>( ScriptStatus);
            Script.and( ScriptStatus == 0, delegate { // если скрипт не загружен, то загружаем его
                
                Script.load_external_script<EXT>();
                Script.and( Script.is_external_script_loaded<EXT>(), delegate {
                    Script.start_new_external_script<EXT>(); // запускаем скрипт
                } );
                
            } );
            
        }, delegate { Script.remove_references_external_script<EXT>(); } ); // выгружаем сведения о скрипте
        jump( LOOP );
    }
    
}
```

Только классы, описывающие внешние скрипты, могут быть переданы в метода **`start_new_external_script`** между символами `<` и `>`. Всё остальное сделает за нас генератор. Мы можем передать туда параметры, если нужно. Давайте посмотрим результат:

```
DEFINE EXTERNAL_SCRIPTS 1 // Use -1 in order not to compile AAA script
DEFINE SCRIPT EXT AT @EXT_ERROR_SKIP			// 0

//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'

//...

0002: jump @TEST_LABEL_0

:TEST_LABEL_0
wait 0
if
0256:     player $2 defined
then
0926: $713 = external_script_status 0 (EXT)
00D6: if
0038:     $713 == 0 // $ == ? (int)
then
08A9: load_external_script 0 (EXT)
00D6: if
08AB:     external_script 0 (EXT) loaded
then
0913: run_external_script 0 (EXT)
end
end
else
090F: end_external_script 0 (EXT)
end
0002: jump @TEST_LABEL_0

//------------- EXTERNAL SCRIPT: 0 (EXT) ---------------
:EXT_ERROR_SKIP
03A4: name_thread 'EXT'

:EXT
00D6: if
8118: not actor $3 dead
then
05BE: AS_actor $3 die
end
004E: end_thread
```

Скрипт конечно бесполезный, но как его загрузить Вы уже знаете.

{% hint style="info" %}
Некоторые методы работают без загрузки скрипта.
{% endhint %}
