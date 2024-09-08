# Генератор парковки

Создавать парковочный транспорт можно через свойство `CAR_PARK`. Нам необходимо инициализировать переменную транспорта и передать её в один из доступных методов. Передать мы можем только переменные с глобальным контекстом. В противном случае будет вызвана ошибка. Методы `init` и `init_with_number_plate` имеют внутренний счётчик, который контролируют лимиты. Вот они:

| Описание              | III | VC  | SA  |
| --------------------- | --- | --- | --- |
| Количество элементов: | 160 | 185 | 500 |

Методы создания парковочного транспорта также были упрощены, и теперь нам будет достаточно этого:

```csharp
public class TEST : Thread {

    static Car parkingCar;
    static Boat parkingBoat;

    public override void START( LabelJump label ) {

        CAR_PARK.init( parkingCar,  0.0,   21.0,   112.0, 180.0, CarModel.ADMIRAL );
        CAR_PARK.init( parkingBoat, 120.0, -231.0, 13.0,  90.0,  BoatModel.JETMAX );

        end_thread();
    }

}
```

Если нужно указать конкретные параметры, то вместо `null` нужно написать нужное значение:

```csharp
public class TEST : Thread {

    static Heli parkingHeli;

    public override void START( LabelJump label ) {

        // null: пропустить параметр (см. подсказки Visual Studio)
        CAR_PARK.init( parkingHeli, 1440.0, 12.0,  12.9,  30.0,  HeliModel.POLMAV, 0, 1, null, null, 4 );

        end_thread();
    }

}
```

В итоге мы получит такой код для Sanny Builder:

```
//------------- THREAD TEST ---------------
:TEST
03A4: name_thread 'TEST'
014B: $2000 = init_car_generator 497 color 0 1 force_spawn 0 alarm 0 door_lock 4 min_delay 0 max_delay 10000 at 1440.0 12.0 12.9 angle 30.0
004E: end_thread
```

{% hint style="warning" %}
Метод **init\_with\_number\_plate** доступен только в библиотеке **GTA.SA**!
{% endhint %}
