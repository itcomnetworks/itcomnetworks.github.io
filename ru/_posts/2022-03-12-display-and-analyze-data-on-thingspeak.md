---
layout: post
title:  "IoT-Ep3 | Отображение и анализ данных на Thingspeak"
date:   2022-03-12 00:00:00 +0300
category: Tech
image: /tech-blog/assets/ThingspeakAccount.png
tags: [IoT, Thingspeak]
lang: ru
lang-ref: Display and analyze data on Thingspeak
---
В этой инструкции мы изучим особенности настройки графика данных и используем язык Matlab для написания скриптов обработки данных на Thingspeak. Вы можете посмотреть демо-видео или инструкцию ниже.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/Xu68phWUE6Q" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Section1)
2. [Настройка графика](#Section2)
3. [Приложения Apps](#Section3)
4. [Создание диаграмму распределения](#Section4)
5. [Добавление измерительных часов](#Section5)
6. [Заключение](#Section6)
7. [Дополнительные источники](#Section7)

## 1. Введение <a name="Section1"></a>
В предыдущей статье IoT-Ep2 мы увидели, как создать канал на Thingspeak для хранения данных о температуре и влажности, полученных с датчика DHT22. С данными, загруженными из модуля NodeMCU, мы получаем 2 графика температуры и влажности от времени.
В этой статье мы рассмотрим некоторые функции Thingspeak, например:
- Настройка отображения графика.
- Добавление других типов аналитических диаграмм.
- И расмотрение дополнительных приложений.

## 2. Настройка графика <a name="Section2"></a>
С каждым графиком, Thingspeak позволяет нам настроить отображение прямо на веб-сайте. Чтобы открыть окно настройки графика, мы щелкним значок карандаша над окном графика (Рис. 1).
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/EditBieuDo.png" style="zoom:50%">
  <br>
    <em>Рисунок 1. Выбор настройки диаграммы</em>
</p>

Получив окно настройки (Рис. 2), мы можем заполнить информацию и настроить график. Например, название графика температуры может быть **График температуры**. Далее мы также можем переименовать оси **Ох** - **Время** и **Оу** - **Температура, °С**. В разделе настройки мы также можем настроить отображение среднего значения или суммы значений за период обзора. Кроме того, нам можно выбрать диапазон значений температуры от 0 до 60.
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/FieldOptions.png" style="zoom:40%">
  <br>
    <em>Рисунок 2. Выборы отображения графика</em>
</p>

Аналогично графику влажности после реадактирования настройки получаем Рис. 3.
<p align="center">
  <img alt="Graph" src="/tech-blog/assets/DoThi.png" style="zoom:40%">
  <br>
    <em>Рисунок 3. Графики температуры и влажности</em>
</p>

### 3. Приложения Apps<a name="Section3"></a>
Кроме того, Thingspeak также позволяет нам создавать различные приложения (Рис. 3).
<p align="center">
  <img alt="Apps" src="/tech-blog/assets/ThingspeakApps.png" style="zoom:40%">
  <br>
    <em>Рисунок 4. Дополнительные приложения от Thingspeak</em>
</p>
Приложения разделены на 2 группы на Thingspeak (Рис. 5): Аналитика (Analytics) и Действия (Actions):
- Группа приложений **Analytics** позволяет нам писать сценарии по языку Matlab для анализа и построения графиков, в этой группе Thingspeak также позволяет создавать плагины, написанные с помощью HTML, CSS и Javascript для отображения на браузере.
- Группа **Actions** позволяет создавать команды запроса, которые помогут нам установить связь с другими приложениями, такими как Twitter, или просто команду HTTP-запроса для удаленного управления устройством.
- Анализ Matlab позволяет нам выполнять код на языке Matlab непосредственно в облаке. Используя Matlab, мы можем анализировать, обрабатывать данные датчиков и строить модели анализа данных. Скрипты Matlab связаны с группой **Actions**, отсюда мы можем установить, с каким периодом скрипт вызывается для выполнения или выполняется при получении запроса.
- Визуализация Matlab позволяет нам использовать язык Matlab для рисования графиков или графиков данных.
- Используя плагины, мы можем создавать диаграммы с помощью HTML, CSS и JavaScript. Оттуда мы можем встроить любой плагин на свой сайт.
- ThingTweet помогает связать устройство с учетной записью Twitter и позволяет отправлять сообщения.
- TimeControl позволяет нам устанавливать время выполнения команд с помощью Matlab Analysis, ThingTweet и TalkBack.
- React и TalkBack помогают нам создавать логику действий на основе полученных входных данных.
- ThingHTTP позволяет нам создавать каналы связи между устройствами, веб-сайтами и веб-службами без необходимости реализации протокола на уровне устройства. Здесь мы можем привязываться к действиям с помощью других приложений ThingSpeak, таких как TweetControl, TimeControl и React.

<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/ThingspeakApps2.png" style="zoom:50%">
  <br>
    <em>Рисунок 4. Группы Analytics và Actions</em>
</p>

### 4. Создание диаграммы распределения <a name="Section4"></a>
Возвращаясь к **private view** канала, мы будем использовать **Matlab Visualization** для построения графика распределения температуры и влажности. Чтобы создать гистограмму, мы перейдем к **Matlab Visualization** (Рис. 5) и выберем пример кода для построения гистограммы (Рис. 6).
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization.png" style="zoom:50%">
  <br>
    <em>Рисунок 5. Выбор Matlab Visualization</em>
</p>

<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization2.png" style="zoom:40%">
  <br>
    <em>Рисунок 6. Создание графика распределения</em>
</p>

После нажатия **Create** у нас будет окно написания кода Matlab (Рис. 7). В этом скрипте мы можем назвать **Temperature Distribution**, а также нам нужны **Channel ID**, **Read API Key**, **fieldID**. В правой части экрана у нас есть информация о канале **Channel**, чтобы программа могла получать доступ для записи и чтения данных. Кроме того, в разделе **Display Settings** мы можем выбрать, какой канал отображать график и отображать его как **Public** или **Private**.
<p align="center">
  <img alt="Edit Matlab" src="/tech-blog/assets/MatlabVisualization3.png" style="zoom:50%">
  <br>
    <em>Рисунок 7. Редактирование кода Matlab</em>
</p>

После нажатия **Save and Rung**, гистограмм отображается (Рис. 8).
<p align="center">
  <img alt="Edit Matlab" src="/tech-blog/assets/MatlabVisualization4.png" style="zoom:30%">
  <br>
    <em>Рисунок 8. Запуск скрипта Matlab</em>
</p>

### 5. Добавление измерительных часов <a name="Section5"></a>
Далее, чтобы создать часы, отображающие температуру, мы можем добавить их, перейдя в раздел **Add Widgets** и выбрав шаблон отображения часов **Gauge** (Рис. 9-11).
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization5.png" style="zoom:30%">
  <br>
    <em>Рисунок 9. Добавление Widgets</em>
</p>
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization6.png" style="zoom:30%">
  <br>
    <em>Рисунок 10. Создание Gauge</em>
</p>
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization7.png" style="zoom:30%">
  <br>
    <em>Рисунок 11. Настройка Widgets</em>
</p>

Таким образом, в **Private view** мы увидим гистограмму распределения и измерительные часы для значения температуры (Рис. 12).
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/MatlabVisualization8.png" style="zoom:30%">
  <br>
    <em>Рисунок 12. Гистограмма распределения и измерительные часы</em>
</p>

### 6. Заключение <a name="Section6"></a>
Можно сказать, что использование Matlab позволяет нам легко настраивать анализ и обработку данных, а также рисовать графики данных.
В настоящее время Matlab является одной из популярных платформ и языков для вычислительных приложений и используется во многих различных областях. Кроме того, использование Matlab также позволяет создавать модели машинного обучения или нейронные сети для прогнозирования изменения данных в будущем.
Кроме того, мы также можем видеть, что рисование и настройка графиков также легко поддерживаются на Thingspeak и позволяют интегрироваться в другие веб-сайты.

### 7. Дополнительные источники <a name="Section7"></a>
- Adruino IDE: [https://www.arduino.cc](https://www.arduino.cc/){:target="_blank"}
- ESP8266-Arduino: [https://github.com/esp8266/Arduino](https://github.com/esp8266/Arduino){:target="_blank"}
- DHT Sensor Library: [https://github.com/adafruit/DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library){:target="_blank"}