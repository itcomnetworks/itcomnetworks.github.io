---
layout: post
title:  "IoT-Ep 1 | Программирование NodeMCU с DHT22 на Arduino"
date:   2022-03-07 00:00:00 +0300
category: Tech
image: /tech-blog/assets/industry-2630319_1280.jpg
comments: true
tags: [IoT, NodeMCU, DHT22, Arduino]
lang: ru
lang-ref: IoT-Ep1 | NodeMCU+DHT22 read temperature and humidity
---

В данной статье мы познакомимся с модулем NodeMCU и датчиком DHT22 для получения температуры и влажности. Вы можете посмотреть демо видео или подробно в интрукции внизу.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/EfuIxwY--uk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Введение)
2. [Установка Arduino и библиотеки программирования ESP8266](#Section2)
3. [Библиотека датчика DHT22](#Section3)
4. [Схема соединения и программирования](#Section4)
5. [Заключение](#Section5)
6. [Дополнительные источники](#Section6)

### 1. Введение <a name="Введение"></a>
В настоящее время, появляется большое количество приложений Интернета вещей (ИВ). Задача удаленного мониторинга окружающей среды может быть решена при применении беспроводных сенсорных сетей. Помимо этого, развитие инфо-телекоммуникационных систем способствовало развитию множество различных типов услуг в нашей жизни.

Данная статья, посвященная основам ИВ, предоставляет простое приложение с использованием модуля NodeMCU и датчика DHT22 для сбора температуры и относительной влажности. Мы будем рассматривать следующее:
- Как установить среду программирования Arduino.
- Как установить библиотеку использования датчика DHT22.
- Как программировать NodeMCUдля сбор данных датчика DHT22.

**Что такое NodeMCU?** NodeMCU (Рис. 1) -- отладочная плата, разработанная на основе чипа WiFi ESP8266, используется для создания устройств Интернета вещей. Модуль NodeMCU, оснащенный чипом WiFi, может отправлять и получать информацию через WiFi в локальную сеть или Интернет. В настоящее время, NodeMCU является одним из дешевых и популярных отладочных плат для разработчиков умных приложения ИВ.
<p align="center">
  <img alt="NodeMCU" src="/tech-blog/assets/NodeMCU.jpeg" style="zoom:80%">
  <br>
    <em>Рисунок 1. Карта пинов (pinout) NodeMCU</em>
</p>

Далее, мы рассмотрим как использовать среду программирования Arduino для программирования NodeMCU для чтения данных датчика DHT22.

## 2. Установка Arduino и библиотеки программирования ESP8266 <a name="Section2"></a>
Сначало, нам нужно программное обеспечение, библиотеки программирования и программные аппараты.
- [Arduino IDE](https://www.arduino.cc/) -- среда программирования для встроенных устройств, таких как Arduino, ESP8266 (NodeMCU), ESP32. Arduino является программой с открытым исходным кодом, которая упрощаяет написание кода и загрузка программ в модуль управления. Вы можете скачать Arduino IDE по [данной ссылке](https://www.arduino.cc/en/software).
- [ESP8266-Arduino](https://github.com/esp8266/Arduino) -- библиотека программирования ESP8266 в среде программирования Arduino.

Для установки библиотеки ESP8266, нам нужно открыть **Preferences** на Arduino IDE. Далее, вставить ссылку ```https://arduino.esp8266.com/stable/package_esp8266com_index.json``` в поле управления URLs через **Preferences -> Additional Boards Manager URLs** (Рисунок 2).
<p align="center">
  <img alt="Arduino Preferences" src="/tech-blog/assets/Arduino.png" style="zoom:70%">
  <br> 
    <em>Рисунок 2. Окно "Preferences" на Arduino</em>
</p>

Далее, мы открываем окно управления устройствам через **Tools --> Board --> Board Manager** (Рисунок 3). Вводим **ESP8266** в поиск, потом нажать **Install** для установки библиотеки программирования ESP8266, включая NodeMCU. После установки, нажимаем **Close** для закрытия окна.
<p align="center">
  <img alt="Board Manager" src="/tech-blog/assets/BoardsManage.png" style="zoom:70%">
  <br>
    <em>Рисунок 3. Окно управления устройствам</em>
</p>

Сейчас, вы может выбрать нашу плату через **Tools --> Board --> ESP8266 Boards --> NodeMCU 0.9 hoặc 1.x** (Рисунок 4). Таким образом, Arduino IDE была установлена и готовка к программирования для модуля NodeMCU.
<p align="center">
  <img alt="Choose NodeMCU" src="/tech-blog/assets/ChooseBoard.png" style="zoom:60%">
  <br>
    <em>Рисунок 4. Выбор платы</em>
</p>

### 3. Библиотека датчика DHT22<a name="Section3"></a>
Далее мы можем использовать библиотеку для использования датчика DHT22. На среде Arduino IDE, мы тоже можем управлять необходимыми библиотеками (скачать или удалить). Чтобы добавить библиотеку нам нужно перейти через **Sketch --> Include Library --> Manage Libraries** (Рисунок 5).
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/ManageLib.png" style="zoom:70%">
  <br>
    <em>Рисунок 5. Управление библиотеками</em>
</p>

В поиске, мы вводим **DHT sensor**, потом поищем название **DHT sensor library**, разработанной **Adafruit**. После этого, нажимаем **Install** для уставноки библиотеки (Рисунок 6). Нам нужно выбрать **Install all** для установки еще зависимых библиотек.
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/LibManager.png" style="zoom:70%">
  <br>
    <em>Рисунок 6. Окно управления библиотекам</em>
</p>

### 4. Схема соединения и программирования <a name="Section4"></a>
Можно соединять датчик DHT22 с модулем NodeMCU по схеме на Рисунке 8. Мы можем соединить следующим образом:
- Пин **VCC** датчика соединяется с пином **3V3** модуля NodeMCU.
- Пин **GND** датчика соединяется с пином **GND** модуля NodeMCU.
- Пин **Data** датчика соединяется с пином **D2** модуля NodeMCU.

<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/DHT11-DHT22-AM2302-Temperature-Humidity-Sensor-Pinout.png" style="zoom:70%">
  <br>
    <em>Рисунок 7. Схема пинов датчиков DHT11 и DHT22</em>
</p>

<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/NodeMCU_DHT.png" style="zoom:50%">
  <br>
    <em>Рисунок 8. Схема соединения NodeMCU с DHT22</em>
</p>

Возвращая в окно Arduino IDE, мы можем открыть один пример от библиотеки DHT Sensor. Мы открываем пример через **File --> Examples --> DHT Sensor library --> DHTtester** (Рисунок 9). Данный пример используется для тестирования датчика.
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/DHTSketchExp.png" style="zoom:50%">
  <br>
    <em>Рисунок 9. Пример тестирования DHTTester</em>
</p>

В данном коде, мы ищем строку **#define DHTPIN** для объявления того, что пин данных датчика соединяется с каким пином модуля NodeMCU. В данном примере, мы соединяем пин данных датчика с пином **D2** (Рисунок 10).
<p align="center">
  <img alt="DHT tester" src="/tech-blog/assets/DHT_sketch.png" style="zoom:50%">
  <br>
    <em>Рисунок 10. Пример считывания данных датчика</em>
</p>
Для загрузки программы в микроконтроллер, сначало нам нужно проверить модуль соединяется с компьюетром через какой порт. Мы открываем через **Tools --> Port --> Serial ports**  (Рисунок 11).
<p align="center">
  <img alt="COM Port" src="/tech-blog/assets/COMPort.png" style="zoom:50%">
  <br>
    <em>Рисунок 11. Выбор порта serial</em>
</p>

Для проверки кода, мы можем нажать **Verify** для компиляции программы. Потом мы можем нажать **Upload** для загрузки программы (Рисунок 10). После завершения загрузки программы мы нажимаем **Serial Monitor** (Рис. 10), чтобы увидеть информацию датчика, напечатанную на экране.
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/SerialMonitor.png" style="zoom:50%">
  <br>
    <em>Рисунок 12. Окно мониторинга порта -- Serial Monitor</em>
</p>

### 5. Заключение <a name="Section5"></a>
В данной инструкции мы завершили шаги по установке программного и аппаратного обеспечения для использования модуля NodeMCU с датчиком DHT22 для измерения температуры и влажности.

Исходя из этого примера, мы можем развивать дальше, например, отправлять полученные данные в **Облако**, к которым мы можем получить доступ и контролировать удаленно через Интернет. Например, сейчас существует множество вычислительных платформ, таких как Thingspeak, Dashboards, MS Azure, Amazon Web Service, которые поддерживают управление и хранение данных с устройств Интернета вещей.

### 6. Дополнительные источники <a name="Section6"></a>
- Arduino IDE: [https://www.arduino.cc/](https://www.arduino.cc/){:target="_blank"}{:target="_blank"}
- ESP8266-Arduino: [https://github.com/esp8266/Arduino](https://github.com/esp8266/Arduino){:target="_blank"}{:target="_blank"}
- DHT Sensor Library: [https://github.com/adafruit/DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library){:target="_blank"}{:target="_blank"}