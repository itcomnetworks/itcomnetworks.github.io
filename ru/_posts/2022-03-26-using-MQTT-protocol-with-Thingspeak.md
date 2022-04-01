---
layout: post
title:  "IoT-Ep6 | Использование протокола MQTT с Thingspeak"
date:   2022-03-26 00:00:00 +0300
category: Tech
image: /tech-blog/assets/MQTT.svg
tags: [IoT, Thingspeak, MQTT]
lang: ru
lang-ref: using mqtt protocol with Thingspeak
---
В этой статье мы рассмотрим протокол MQTT для обновления данных датчика на Thingspeak.
Вы можете посмотреть демо-видео или инструкцию ниже.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/VQEWx5P10uM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Section1)
2. [Протокол MQTT и платформа Thingspeak](#Section2)
- [2.1. Протокол MQTT](#Subsection2.1)
- [2.2. Испольование MQTT с Thingspeak](#Subsection2.2)
3. [Соединение и программирования](#Section3)
4. [Заключение](#Section4)
5. [Дополнительные источники](#Section5)

### 1. Введение <a name="Section1"></a>
В настоящее время протокол HTTP(s) используется для передачи информации в большинстве веб-приложений. Однако сегодняшнее технологическое развитие приносит больше различных приложений и устройств, где требуются малая задержка и низкое энергопотребление. Большинство этих устройств относятся к группам приложений Интернета вещей (ИВ). Начиная с первой версии в 1996 году и до сих пор протокол HTTP широко использовался в большинстве приложений, использующих веб-браузеры. Но, возможно, в этом нет необходимости с размером **заголовков** в HTTP-пакете, когда устройству ИВ требуется меньше ресурсов для отправки сообщения на удаленный сервер.

Чтобы уменьшить количество служебных пакетов и размер **заголовков** в протоколе HTTP, был разработан другой протокол под названием **MQTT** (Message Queuing Telemetry Transport), который позволяет устройствам обмениваться сообщениями друг с другом и потреблять меньше сетевых ресурсов.

### 2. Протокол MQTT и платформа Thingspeak <a name="Section2"></a>
#### 2.1. Протокол MQTT <a name="Subsection2.1"></a>
MQTT — это протокол, основанный на модели **Публикация/Подписка** (Publish/Subscribe), разработанный для устройств с ограниченной мощностью и пропускной способностью для передачи по беспроводным сетям. Предполагается, что это простой протокол, работающий на **TCP/IP sockets** или **WebSockets**. Как и HTTP, протокол MQTT можно защитить с помощью протокола TLS/SSL. Модель **Публикация/Подписка** позволяет непрерывно доставлять сообщения другим клиентам без отправки частых запросов на сервер. В отличие от модели **Запрос/Ответ** по протоколу HTTP, клиентам, которые хотят получать информацию, необходимо **Подписаться** (Subscribe) на топик, чтобы следить за топиком на **Брокере** (Broker). На Hис. 1 изображен простая схема взаимодействия с устройствами по протоколу MQTT.
- **Брокер**: это сервер или устройство, которое пересылает сообщения между устройствами на основе **топиков**, на которые устройство **публикует** (publish) или **подписывается** (subcribe) этот **топик**.
- **Издатель**: это устройство, которое отправляет сообщение в отдельный **топик**. Предположим, что издатель NodeMCU будет отправлять данные о температуре и влажности в топик «/topic_weather».
- **Подписчик**: это устройство, которое хочет получать сообщения от других устройств через **Брокер**. Если это устройство хочет отслеживать информациям о температуре и влажности от модуля NodeMCU, ему необходимо **подписаться** на топик «/topic_weather». После успешной регистрации, каждый раз когда NodeMCU отправляет новое сообщение, **Брокер** будет немедленно отправлять его на устройства, подписавшиеся на топик «/topic_weather».

<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MQTT.svg" style="zoom:50%">
  <br>
    <em>Рисунок 1. Модель взаимодействия по протоколу MQTT</em>
</p>

Кроме того, исходящие сообщения MQTT можно настроить так, чтобы они соответствовали требованиям QoS (качество обслуживания). Мы будем рассматривать больше о протоколе MQTT в отдельном посте.


#### 2.2. Использование протокола MQTT c Thingspeak <a name="Subsection2.2"></a>
ThingSpeak предоставляет URL-адрес Брокера по адресу **mqtt3.thingspeak.com**. Конфигурация для Клиента, который хочет подключиться по MQTT, может быть следующей:

|Порт |Тип подключения |Шифрование |
|---|----|-----|
|1883   | TCP   | Нет |
|8883   | TCP   | TLS/SSL   |
|80      | Websocket | Нет |
|443     | Wbsocket     | TLS/SSL   |

Есть 2 способа **MQTT Publish** для обновления данных на **Channel** Thingspeak.

| Публикация MQTT | Топик | Полезная нагрузка |
| Публикация в многоразные поля одновремено  | channels/\<channelID\>/publish | field1=\<value1\>&field2=\<value2\> |
| Публикация одного значения в одное поле | channels/\<channelID\>/publish/fields/field\<fieldNumber\> | \<value\>

В данной статье мы будем отправлять температуру и влажность в поле **field1** и **field2** одновременно. Чтобы подключиться к Thingspeak по протоколу MQTT, нам нужно создать MQTT-устройство. Шаги для инициализации устройства изображены следующим образом:

**1.** Согласно Рис. 2, мы выбираем **Devices --> MQTT** в меню сайта.
<p align="center">
  <img alt="MQTT Devices" src="/tech-blog/assets/Thingspeak_MQTT01.png" style="zoom:50%">
  <br>
    <em>Рисунок 2. Создание устройства MQTT</em>
</p>

**2.** Выбираем **Add a new device** для добавление нового устройства. Затем, нам нужно заполнить основные информации для нового устройство, и разрешить ему доступ к каналу (**Channel**),  в котором мы хотим обновить данные (Рис. 3).
<p align="center">
  <img alt="MQTT Devices" src="/tech-blog/assets/Thingspeak_MQTT02.png" style="zoom:50%">
  <br>
    <em>Рисунок 3. Настройка устройства MQTT</em>
</p>

**3.** После нажатия кнопки **Add Device** мы получаем окно, в котором сообщается, что у нас есть учетные данные MQTT для публикации и подписки на Thingspeak. Здесь мы можем **Скачать** файл **mqtt_secrets.h** для программирования на Arduino (Рис. 4).
<p align="center">
  <img alt="MQTT Devices" src="/tech-blog/assets/Thingspeak_MQTT03.png" style="zoom:50%">
  <br>
    <em>Рисунок 4. Учетные данные для устройства MQTT</em>
</p>

### 3. Соединение и программирования <a name="Section3"></a>
В данной статье, мы продолжаем работать с NodeMCU и DHT22. Датчик можно соединить с модулем по схеме на Рис. 5.
<p align="center">
  <img alt="NodeMCU+DHT22" src="/tech-blog/assets/NodeMCU_DHT.png" style="zoom:40%">
  <br>
    <em>Рисунок 5. Схема соединения NodeMCU и DHT22</em>
</p>

Далее, мы открываем среду программирования **Arduino IDE** для написания кода. Сначало, нам нужно перейти в раздел управления библиотеками через **Tools --> Manage Libraries**. Затем найдем библиотеку **EspMQTTClient** и установим ее и зависимые библиотеки (Рис. 6).
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/Thingspeak_MQTT04.png" style="zoom:50%">
  <br>
    <em>Рисунок 6. Установка библитеки EspMQTTClient</em>
</p>

Вы можете найти пример кода [Здесь](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}

Или можно скопировать следующий код и вставить в Arduino IDE.
{% highlight C %}
#include "EspMQTTClient.h"
#include "mqtt_secrets.h"
#include "DHT.h"

EspMQTTClient client(
  SECRET_WIFI_NAME,
  SECRET_WIFI_PASSWORD,
  "mqtt3.thingspeak.com",  // MQTT Broker server ip
  SECRET_MQTT_USERNAME,   // Can be omitted if not needed
  SECRET_MQTT_PASSWORD,   // Can be omitted if not needed
  SECRET_MQTT_CLIENT_ID      // Client name that uniquely identify your device
);

// DHT information
#define DHTPIN D2
#define DHTTYPE DHT22       // or set DHT11 type if you use DHT11
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastTime = 0;
unsigned long delayTime = 20000; // set a period of sending data.

void setup() {
  Serial.begin(115200);

  // Begin DHT
  dht.begin();
}

void onConnectionEstablished() {
  Serial.println("MQTT Client is connected to Thingspeak!");
}

void publishData() {
  if (client.isConnected() && (millis() - lastTime > delayTime)) {
    float h = dht.readHumidity();
    // Read temperature as Celsius (the default)
    float t = dht.readTemperature();
  
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      return;
    }
    Serial.print(F("\nTemperature: "));
    Serial.print(t);
    Serial.print(F("°C "));
    Serial.print(F("Humidity: "));
    Serial.print(h);
    Serial.println(F("%  "));

    Serial.println(F("\nPublising data to Thingspeak"));
    
    String MY_TOPIC = "channels/" + String(CHANNEL_ID) + "/publish";  // build your topic: channels/<channelID>/publish
    String payload = "field1=" + String(t) + "&field2=" + String(h);
    client.publish(MY_TOPIC, payload);

    lastTime = millis();
  }
}

void loop() {
  client.loop();
  publishData();
} 
{% endhighlight %}

После сохранения файла Arduino на диске, нам нужно будет переместить файл **mqtt_secretss.h** в каталог, где находится только что сохраненный файл Arduino. Или вы также можете создать в проекте дополнительный файл (Рис. 7) и назвать его **mqtt_secrets.h**.
<p align="center">
  <img alt="MQTT Devices" src="/tech-blog/assets/Thingspeak_MQTT05.png" style="zoom:50%">
  <br>
    <em>Рисунок 7. Создание дополнительного файла на Arduino</em>
</p>
В файле **mqtt_secrets.h** нам будет нужно редактировать следующие информации.
{% highlight C %}
#define SECRET_MQTT_USERNAME "YOUR_MQTT_USERNAME"  
#define SECRET_MQTT_CLIENT_ID "YOUR_MQTT_CLIENT_ID"  
#define SECRET_MQTT_PASSWORD "YOUR_MQTT_PASSWORD"
#define SECRET_WIFI_NAME "YOUR_WIFI_NAME"
#define SECRET_WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
#define CHANNEL_ID "YOUR_CHANNEL_ID"
{% endhighlight %}

**Давайте посмотрим на значение приведенных выше строк кода..**

Сначала нам нужно включить библиотеки для использования WIFI, MQTT и датчика DHT.
{% highlight C %}
#include "EspMQTTClient.h"
#include "mqtt_secrets.h"
#include "DHT.h"
{% endhighlight %}

Затем создаем экземпляр объекта **client**, который позволяет управлять подключениями, включая подключения WiFi, подключение по MQTT с использованием библиотеки **ESPMQTTClient.h**.
{% highlight C %}
EspMQTTClient client(
  SECRET_WIFI_NAME,
  SECRET_WIFI_PASSWORD,
  "mqtt3.thingspeak.com",  // MQTT Broker server ip
  SECRET_MQTT_USERNAME,   // Can be omitted if not needed
  SECRET_MQTT_PASSWORD,   // Can be omitted if not needed
  SECRET_MQTT_CLIENT_ID      // Client name that uniquely identify your device
);
{% endhighlight %}

В дополнение к основным функциям у нас есть функция **onConnectionEstablished()**, которая вызывается после успешного подключения к Thingspeak.
{% highlight C %}
void onConnectionEstablished() {
  Serial.println("MQTT Client is connected to Thingspeak!");
}
{% endhighlight %}

Далее есть функция **publishData()**, которая позволяет отправлять данные каждые 20 секунд.
{% highlight C %}
void publishData() {
  if (client.isConnected() && (millis() - lastTime > delayTime)) {
    float h = dht.readHumidity();
    // Read temperature as Celsius (the default)
    float t = dht.readTemperature();
  
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      return;
    }
    Serial.print(F("\nTemperature: "));
    Serial.print(t);
    Serial.print(F("°C "));
    Serial.print(F("Humidity: "));
    Serial.print(h);
    Serial.println(F("%  "));

    Serial.println(F("\nPublising data to Thingspeak"));
    
    String MY_TOPIC = "channels/" + String(CHANNEL_ID) + "/publish";  // build your topic: channels/<channelID>/publish
    String payload = "field1=" + String(t) + "&field2=" + String(h);
    client.publish(MY_TOPIC, payload);

    lastTime = millis();
  }
}
{% endhighlight %}

В функции **loop()** объект **client** вызывается с помощью **loop()**, чтобы всегда проверять состояния устройства, а функция **publishData()** вызывается для публикации измеренных данные о температуре и влажности в топик.
{% highlight C %}
void loop() {
  client.loop();
  publishData();
}
{% endhighlight %}

После **Загрузки** программы в **NodeMCU** мы можем отслеживать некоторую информацию, распечатываемую на **Serial Monitor**. Наряду с этим данные также обновляются на Thingspeak (Рис. 8).
<p align="center">
  <img alt="MQTT Devices" src="/tech-blog/assets/Thingspeak_MQTT06.png" style="zoom:50%">
  <br>
    <em>Рисунок 8. Окно мониоринга порта - Serial Monitor</em>
</p>

### 4. Заключение <a name="Section4"></a>
В этой статье мы рассмотрели прицип работы протокола MQTT и как обновлять поля данных на **Channel** на Thingspeak.
Сегодня, протокол MQTT считается одним из популярных простых протоколов, используемых для устройств Интернета вещей. Мы также видим, что программирование для **публикации** данных также проще, чем при использовании протокола HTTP.
Тем не менее, есть еще недостатки протокола MQTT, которые не были упомянуты в этой статье, но будут обобщены в другой статье.

### 5. Дополнительные источники <a name="Section5"></a>
- Пример скетча: [https://github.com/itcomnetworks/IoT-Series](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}
- [Wifi and MQTT handling for ESP8266 and ESP32](https://github.com/plapointe6/EspMQTTClient){:target="_blank"}
- [Using MQTT with Thingspeak](https://www.mathworks.com/help/thingspeak/mqtt-basics.html){:target="_blank"}