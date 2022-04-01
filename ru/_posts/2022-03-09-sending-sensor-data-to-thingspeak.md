---
layout: post
title:  "IoT-Ep2 | Отправление данных датчика в облако  Thingspeak"
date:   2022-03-09 00:00:00 +0300
category: Tech
image: /tech-blog/assets/ThingspeakAccount.png
comments: true
tags: [IoT, Thingspeak, NodeMCU, DHT22]
lang: ru
lang-ref: Sending sensor data to Thingspeak
---
В данной иструкции, мы будем рассматривать платформу Thingspeak - одна из популярных платформ Интернета вещей (IoT Platform). Платформа Thingspeak позволяет хранение и управления данными ИВ. Вы можете посмотреть демо-видео или инструкцию ниже.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/jfxlOaFn1a8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Section1)
2. [Платформа Thingspeak](#Section2)
3. [Настройка Thingspeak и поля для хранения данных датчиков](#Section3)
4. [Установка библиотеки Thingspeak для программирования на Arduino IDE](#Section4)
5. [Подключение и программированя NodeMCU](#Section5)
6. [Графики на Thingspeak](#Section6)
7. [Заключение](#Section7)
8. [Дополнительные источники](#Section8)

### 1. Введение <a name="Section1"></a>
В настоящее время непрерывно увеличивается количество приложений и устройств Интернета вещей (ИВ). Параллельно также предлагается строить и развивать системы и платформы управления устройствами, чтобы эффективно управлять большими объемами информации (Big Data) и большим количеством подключенных устройств.

В этой статье мы узнаем:
- Thingspeak: как создавать канал и поля для хранения данных.
- Как утановить библиотеку Thingspeak для Arduino IDE.
- Как программировать NodeMCU (ESP8266) для отправки данных датчика DHT22 (температуры и влажности) в Thingspeak.

### 2. Платформа Thingspeak <a name="Section2"></a>
   
**Платформа Интернета вещей** позволяет устройствам ИВ подключать и хранить данные датчиков, кроме того, платформа позволяет разрабатывать аналитические программы, которые обрабатывают собранные данные, а также отправляют управляющие сообщения на основе запрограммированной логики.

**ThingSpeak** -- это платформа ИВ, которая предоставляет услуги, позволяющие собирать, визуализировать и анализировать потоки данных в реальном времени в облаке. Благодаря интеграции MATLAB в Thingspeak мы можем писать код для выполнения предварительной обработки, анализа или построения графиков данных. Кроме того, на Thingspeak мы также можем установить дополнительную логику сигнализации или отправлять сообщения пользователям, когда происходит ненормальное изменение полученных данных датчика.

На Рисунке 1 изображена простая схема подключение устройство к Thingspeak.
<p align="center">
  <img alt="Thingspeak" src="/tech-blog/assets/Thingspeak.svg" width=500>
  <br>
    <em>Рисунок 1. Схема подключения Thingspeak</em>
</p>

По данной схеме мы можем понять следующим образом:
- NodeMCU <--> WiFi Router <--> Internet <--> Thingspeak

Модуль NodeMCU подключается к Интернету через точку доступа WiFi, и после этого модуль NodeMCU может отправить пакеты данных к Thingspeak. В данной инструкции, мы будем отправлять данные температуры и влажности в облако.

### 3. Настройка Thingspeak и поля для хранения данных датчиков <a name="Section3"></a>
Сначало, нам нужно создать аккаунт на сайте [thingspeak.com](https://thingspeak.com/) для использования сервисов. После входа в Thingspeak, мы нажимаем **Channels**, чтобы перейти к странице создания канала, или вы можете перейти в меню **Channels --> My Channels** (Рисунок 2).
<p align="center">
  <img alt="Thingspeak Account" src="/tech-blog/assets/ThingspeakAccount.png" style="zoom:30%">
  <br>
    <em>Рисунок 2. Thingspeak</em>
</p>

Затем, мы нажимаем **New Channel** для создания канала хранения данных (Рисунок 3).
<p align="center">
  <img alt="New Thingspeak Channel" src="/tech-blog/assets/NewChannel.png" style="zoom:30%">
  <br>
    <em>Рисунок 3. Создание канала на Thingspeak</em>
</p>

Мы начнем с создания нового канала и заполним основную информацию, такую ​​как имя канала, описание и имя для полей (Рис. 4). Поля здесь представляют собой поля данных, полученные от нашего устройства. Здесь **Field 1** предназначено для **Температуры**, а **Field 2** — для **Влажности**. После заполнения основной информации прокручиваем вниз и нажимаем **Save Channel**, чтобы сохранить и инициализировать канал.
<p align="center">
  <img alt="Thingspeak Channel" src="/tech-blog/assets/ChannelSetting.png" style="zoom:30%">
  <br>
    <em>Рисунок 4. Создание канала и поля канала на Thingspeak</em>
</p>

После создания канала мы видим, что генерируются 2 графика, соответствующего температуре и влажности, которые мы установили для 2 полей данных (Рис. 5). Нам потребуются **API Keys** и **Channel ID**, чтобы иметь возможность записывать и читать данные из канала. Мы нажимаем **API Keys** для посмротра ключей
<p align="center">
  <img alt="Thingspeak APIKeys" src="/tech-blog/assets/APIKeys.png" style="zoom:40%">
  <br>
    <em>Рисунок 5. Канал даных Thingspeak</em>
</p>

Чтобы записать данные в канал, нам нужно использовать **Write API Key** (Рис. 6).
<p align="center">
  <img alt="Write APIKeys" src="/tech-blog/assets/WriteAPIKeys.png" style="zoom: 30%">
  <br>
    <em>Рисунок 6. API Keys для записи и чтения данных</em>
</p>

### 4. Установка библиотеки Thingspeak для программированя на Arduino IDE<a name="Section4"></a>
Для упрощения работы по программирования NodeMCU, мы можем использовать библиотеку, разработанную MathWorks для Arduino. На программе Arduino IDE, мы переходим к диспетчеру библиотеки через **Sketch --> Inlude Library --> Library Manager**, затем поищем **Thingspeak**. Мы увидим библиотеку **Thingspeak**, разработанную MathWorks, далее нажимаем **Install** для установки библиотеки (Рисунок 7).
<p align="center">
  <img alt="Thingspeak Channel" src="/tech-blog/assets/Thingspeak_Lib.png" style="zoom:50%">
  <br>
    <em>Рисунок 7. Установка библиотеки Thingspeak</em>
</p>

### 5. Подключение и программирования NodeMCU <a name="Section5"></a>
Как было сказано в начале поста, мы будем использовать NodeMCU и датчик DHT22. Мы можем подключить DHT22 с модулем NodeMCU по схеме на Рисунке 8.
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/NodeMCU_DHT.png" style="zoom:50%">
  <br>
    <em>Рисунок 8. Схема соединения NodeMCU с DHT22</em>
</p>

Кроме того, библиотека датчика **DHT** тоже используется в данном проекте. Если **DHT Sensor Library** не была установлена, то мы можем перейти к диспетчеру библиотек через **Sketch --> Inlude Library --> Library Manager**, затем поищем **DHT Sensor** (Рисунок 9).
<p align="center">
  <img alt="DHT Lib" src="/tech-blog/assets/LibManager.png" style="zoom:50%">
  <br>
    <em>Рисунок 9. Установка библиотеки DHT Sensor</em>
</p>

Вы можете найти пример кода [здесь - WriteMultipleFieldsSecure](https://github.com/itcomnetworks/IoT-Series/tree/main/WriteMultipleFieldsSecure). После открытия кода в Arduino IDE, мы редактируем данные в файле **secrets.h**. В данном файле, нам необходимо редактировать:
- SECRET_SSID - название  WiFi подключения.
- SECRET_PASS - пароль WiFi.
- SECRET_CH_ID - идентификатор канала (Рис. 5).
- SECRET_WRITE_APIKEY - API для записи данных (Рис. 6).
<p align="center">
  <img alt="Secret File" src="/tech-blog/assets/SecretKeys.png" style="zoom:50%">
  <br>
    <em>Рисунок 10. Редактирования данных подключения</em>
</p>

После завершения редактирования, мы можем нажать **Verify** для компиляция программы, или **Upload** для загрузки прогамммы в NodeMCU. Затем мы тоже можем открыть мониторинг порта **Serial Monitor** для просмотра данных, распечанных в экране с использованием **Serial.print** (Рис. 11).
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/Thingspeak_SerialMonitor.png" style="zoom:50%">
  <br>
    <em>Рисунок 11. Мониторинг порта -- Serial Monitor</em>
</p>
  
### 6. Графики на Thingspeak <a name="Section6"></a>
Вернувшись на экран Thingspeak с вкладкой **Private View**, мы увидим 2 графика изменения данных, соответствующих измеренным данным с датчика DHT22.
<p align="center">
  <img alt="Graph" src="/tech-blog/assets/Thingspeak_BieuDo.png" style="zoom:50%">
  <br>
    <em>Рисунок 12. Графики отображения данных датчика DHT22</em>
</p>

### 7. Заключение <a name="Section7"></a>
В этой статье мы узнали, как отправлять данные температуры и влажности с датчика DHT22 на платформу Thingspeak с помощью модуля NodeMCU WiFi. Кроме того, на Thingspeak мы также можем настраивать отображение графиков данных и писать скрипты в Matlab для анализа данных.

### 8. Дополнительные источники <a name="Section8"></a>
- Пример скетча: [https://github.com/itcomnetworks/IoT-Series](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}
- Thingspeak: [https://thingspeak.com/](https://thingspeak.com/){:target="_blank"}
- thingspeak-arduino: [https://github.com/mathworks/thingspeak-arduino](https://github.com/mathworks/thingspeak-arduino){:target="_blank"}
- ESP8266/Arduino: [https://github.com/esp8266/Arduino](https://github.com/esp8266/Arduino){:target="_blank"}
- DHT Sensor Library: [https://github.com/adafruit/DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library){:target="_blank"}