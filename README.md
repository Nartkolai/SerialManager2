## Serial Manager 2

* [Google Play](https://play.google.com/store/apps/details?id=kg.delletenebre.serialmanager2)
* [Обсуждение на DRIVE2.RU](https://www.drive2.ru/b/487607019713331355/)
* [Обсуждение на pccar.ru](http://pccar.ru/showthread.php?t=24120)

<img src="https://user-images.githubusercontent.com/3936845/36942953-1ef99710-1faa-11e8-8df8-f1ff00468303.png" width="240">
<img src="https://user-images.githubusercontent.com/3936845/36942952-1ecd2608-1faa-11e8-8fd5-d8d038ce7458.png" width="240">
<img src="https://user-images.githubusercontent.com/3936845/36942951-1ea517d0-1faa-11e8-9528-472ac780381d.png" width="240">
<img src="https://user-images.githubusercontent.com/3936845/36942954-1f2218f2-1faa-11e8-95fb-f2598367999b.png" width="240">


## Возможности
* Подключение:
  * USB
  * Bluetooth
  * WebSocket
  
* Действия при получении команды:
  * Показ уведомлений в виде "плавающего" окна
  * Запуск приложения
  * Эмуляция нажатия клавиш клавиатуры
  * Выполнение консольных (shell) команд
  * Отправка данных на контроллер
  

## Arduino → Serial Manager
Формат отправляемой команды: `<key:value>`

Пример простого скетча для ардуино:
```cpp
int counter = 0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  Serial.println("<test:" + String(counter) + ">");
  counter++;
  delay(3000);
}
```

## Serial Manager → Arduino

### Запуск и завершение соединения
При включенной опции `Отправлять состояния соединения`, на контроллер, при каждом успешном
соединении, будет отправлено сообщение
`kg.serial.manager.connection_established`.

При завершении работы сервиса, на все подключенные контроллеры будет отправлено сообщение
`kg.serial.manager.connection_lost`.

### Датчик света
При включенной опции `Отправлять данные датчика освещённости` на контроллер будут отправляться:
* `light_sensor_value:{value}`, где `{value}` значение датчика в люксах;
* `light_sensor_mode:{mode}`, где `{mode}` число от 0 до 7:
  * 0 - [LIGHT_NO_MOON](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_NO_MOON)
  * 1 - [LIGHT_FULLMOON](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_FULLMOON)
  * 2 - [LIGHT_CLOUDY](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_CLOUDY)
  * 3 - [LIGHT_SUNRISE](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_SUNRISE)
  * 4 - [LIGHT_OVERCAST](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_OVERCAST)
  * 5 - [LIGHT_SHADE](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_SHADE)
  * 6 - [LIGHT_SUNLIGHT](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_SUNLIGHT)
  * 7 - [LIGHT_SUNLIGHT_MAX](https://developer.android.com/reference/android/hardware/SensorManager.html#LIGHT_SUNLIGHT_MAX)

Сообщения датчика освещённости отправляются не чаще одного раза в 3 секунды и только при значительном изменении освещения, т.е. при
смене значения `{mode}`.

### Состояние экрана
При включенной опции `Отправлять состояние экрана` на контроллер будут отправляться:
* `screen:on` при включении экрана
* `screen:off` при выключении экрана

### Действие команды `Отправить данные`
На контроллер будет отправлена строка указанная в опциях команды.
К строке будет применено [форматирвание](https://github.com/delletenebre/SerialManager2#Форматирование-строк).


## Serial Manager → Android
Broadcast Intent'ы:
* При получении команды:
  * Action: `kg.serial.manager.command_received`
  * Extras: `key`, `value`
* При запуске программы:
  * `kg.serial.manager.app_started`
* При запуске сервиса:
  * `kg.serial.manager.started`
* При остановке сервиса:
  * `kg.serial.manager.stopped`
  
  
## Android → Serial Manager
Чтобы отправить данные на контроллер из Вашего приложения, необходимо создать Intent со следующими
параметрами:
* Action: `kg.serial.manager.send`
* Extras:
  * `data` - сообщение, которое необходимо отправить на контроллер
  * `id` - идентификатор конкретной операции, необязательный параметр.
  

Пример (Java):
```java
Intent intent = new Intent("kg.serial.manager.send");
intent.putExtra("id", "1234");
intent.putExtra("data", "message_for_controller");
sendBroadcast(intent);
```
В случае, если был указан параметр `id`, то после обработки команды и попытки отправить данные на
контроллер, Serial Manager создаст следующий Broadcast Intent:
* Action: `kg.serial.manager.send.complete`
* Extras: `id`


## Форматирование строк
Можно использовать следующие значения:
* `%key` - будет заменено на ключ текущей команды;
* `%value` - будет заменено на значение текущее команды;
* `hex2dec(x)`, `dec2hex(x)`, `bin2dec(x)`, `dec2bin(x)` - конвертирование `x` из одной системы
счисления в другую;
* `%{...}` - вместо ` ... ` нужно подставить математическое выражение (формулу). [Описание
доступных операторов и выражений](https://github.com/uklimaschewski/EvalEx#supported-operators).

Например, от контроллера приходят данные о температуре в Фаренгейтах, а нам нужно перевести в
градусы Цельсия. Для этого нужно написать:

`%{round((%value - 32) * (5 / 9), 1)} ºC`

По формуле выше, мы перевели градусы из Ф в С и округлили полученное значение до десятых (до одного
знака после запятой). Т.е. если от контроллера пришла цифра 89, то мы на выходе получим `31.7 ºC`.
При этом текст до и после `%{...}` остаётся без изменений.

Форматирование строк применяется:
* при действии команды `Отправить данные`;
* в уведомлении при распознавании команды;
* в создаваемом Intent'е при распознавании команды.


## Интерфейс
Для удаления команды - свайп влево.

Команды можно перемещать вверх и вниз по списку. Для перемещения - долгий tap на команде, затем (не
отпуская) двигайте вверх или вниз.


## Библиотеки
* [UsbSerial](https://github.com/felHR85/UsbSerial)
* [EvalEx](https://github.com/uklimaschewski/EvalEx)

## Альтернативы
* [Serial Manager](https://github.com/delletenebre/SerialManager)
* [Remote Inputs Manager / Remote steering wheel control](http://forum.xda-developers.com/showthread.php?t=2635159)
