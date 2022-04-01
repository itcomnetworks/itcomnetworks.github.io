---
layout: post
title:  "IoT-Ep5 | HTTP POST отправляет данные в Thingspeak"
date:   2022-03-22 00:00:00 +0300
category: Tech
image: /tech-blog/assets/PostRequest.png
tags: [IoT, Thingspeak, HTTP]
lang: ru
lang-ref: HTTP POST send data to thingspeak
---
В этой статье мы рассмотрим, как использовать метод POST (запрос HTTP POST) для обновления данных датчиков на Thingspeak.
Вы можете посмотреть демо-видео или инструкцию ниже.

<p align="center">
<iframe width="560" height="315" src="https://youtube.com/embed/NMUgRzIFaQk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Section1)
2. [Метод HTTP POST](#Section2)
3. [Соединение и программирование](#Section3)
- [3.1. Соединение](#Subsection3.1)
- [3.2 Использование запроса HTTP POST](#Subsection3.2)
4. [Заключение](#Section4)
5. [Дополнительные источники](#Section5)

### 1. Введение <a name="Section1"></a>
Методы **HTTP-запросов** используются в соответствии с индивидуальными требованиями. Платформа Thingspeak не только поддерживает обновление данных с использованием метода **HTTP GET**, но также поддерживает метод **HTTP POST** для отправки запроса на обновление данных по **каналу**.
При использовании метода **GET** все данные формы кодируются в URL-адресе и добавляются к URL-адресу в качестве параметра запроса. При использовании запроса **POST** данные формы отображаются в теле запроса **HTTP POST**.

В этой статье мы узнаем о методе **HTTP POST**, который используется для обновления данных на Thingspeak, и посмотрим, как запрограммировать NodeMCU для отправки запросов **HTTP POST** с данными о температуре и влажности с датчика DHT22.

### 2. Метод HTTP POST <a name="Section2"></a>
В методе **HTTP POST** отправляемые данные находятся в части тела запроса (**request body**). Согласно документации Thingspeak (https://www.mathworks.com/help/thingspeak/writedata.html), отправка данных методом **HTTP POST** может быть представлена ​​двумя способами. В части тела (**Body**) запроса **Request** наши данные могут быть представлены как **закодированные URL** или как **JSON**.

|Запрос        |POST /update.json HTTP/1.1                                |
|Адрес        |Host: api.thingspeak.com                                   |
|Тип данных   |Content-Type: application/json                             |
|Длина тела    |Content-Length: 59                                         |
|Пустая срока     |                                                           |
|Тело           |{"api_key": "RBDRQF9W9SDPJA6T","field1":10,"field2":30}    |

<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/PostRequest.png" style="zoom:50%">
  <br>
    <em>Рисунок 1. Отправка данных типа application/json с методом **HTTP POST**</em>
</p>

или

|Запрос        |POST /update HTTP/1.1                                |
|Адрес        |Host: api.thingspeak.com                             |
|Тип данных   |Content-Type: application/x-www-form-urlencoded      |
|Длина тела    |Content-Length: 43                                   |
|Пустая строка     |                                                     |
|Тело           |api_key=RBDRQF9W9SDPJA6&field1=10&field2=30          |


<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/PostRequest2.png" style="zoom:50%">
  <br>
    <em>Рисунок 2. Отправка данных типа application/x-www-form-urlencoded с методом **POST**</em>
</p>


### 3. Соединение и программирование<a name="Section3"></a>
#### 3.1. Соединение <a name="Subsection3.1"></a>
В этой статье мы продолжаем работать с NodeMCU и DHT22. Датчик можно подключить к модулю NodeMCU по схеме на Рисунке 3.
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/NodeMCU_DHT.png" style="zoom:50%">
  <br>
    <em>Рисунок 3. Схема соединение NodeMCU с DHT22</em>
</p>

#### 3.2. Использование запроса **HTTP POST** <a name="Subsection3.2"></a>
Существует 2 способа программирования в Arduino IDE для NodeMCU для отправки запроса **HTTP POST** на удаленный сервер:
- Способ 1: мы используем только 1 библиотеку **ESP8266WiFi.h**.
- Способ 2: мы можем использовать комбинацию двух библиотек **ESP8266WiFi.h** и **ESP8266HTTPClient.h**.

Вы можете найти пример кода [здесь](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}.

**a. Использование ESP8266WiFi.h**: вы можете скопировать следующий скетч.
{% highlight C %}
#include <ESP8266WiFi.h>
#include "DHT.h"

// You need to edit with your network credentials.
#define WIFI_NAME "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

// Initialize the client library
WiFiClient client;

// You need to edit with your ThingSpeak information.
#define THING_SPEAK_ADDRESS "api.thingspeak.com"
String writeAPIKey="YOUR_WRITE_API_KEY";

// DHT information
#define DHTPIN D2
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastTime = 0;
unsigned long delayTime = 20000; // set a period of sending data.

void setup()
{
  Serial.begin(115200);
  Serial.println();

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.print("Connected, IP address: ");
  Serial.println(WiFi.localIP());

  // Begin DHT Sensor
  dht.begin();
  
  delay(2000);
}

void loop() {
  if ((millis() - lastTime) > delayTime) {
    HTTPPost();
    lastTime = millis();
  }
}

void HTTPPost() {
  if (client.connect(THING_SPEAK_ADDRESS, 80)) {
    Serial.println("Server connected");

    // Read DHT sensor
    float h = dht.readHumidity();    // Read humidity as %
    float t = dht.readTemperature(); // Read temperature as Celsius (the default)
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t)) {
      Serial.println(F("Failed to read from DHT sensor!"));
      return;
    }

    Serial.println("Sending POST request:");
    // String postData= "api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
    String postData = "{\"api_key\":\"" + writeAPIKey + "\",\"field1\":\"" + String(t) + "\",\"field2\":\"" + String(h) + "\"}";
    // client.println("POST /update HTTP/1.1");
    client.println("POST /update.json HTTP/1.1");
    client.println( "Host: api.thingspeak.com" );
    client.println( "Connection: close" );
    // client.println( "Content-Type: application/x-www-form-urlencoded" );
    client.println( "Content-Type: application/json" );
    client.println( "Content-Length: " + String( postData.length() ) );
    client.println();
    client.println( postData );
    Serial.println( postData );

    unsigned long timeout = millis();
    while (client.available() == 0) {
      if (millis() - timeout > 5000){
        Serial.println(">>> Client Timeout !");
        client.stop(); return;
        }
    }
    
    // Read all the lines of the reply from server and print them to Serial
    while (client.available()){
      String line = client.readStringUntil('\r'); Serial.print(line);
    }
    
    Serial.println("\nClosing connection\n");
  }
}
{% endhighlight %}

Изменение информации о подключение модуля к точке доступа WiFi:
{% highlight C %}
// You need to edit with your network credentials.
#define WIFI_NAME "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
{% endhighlight %}

Затем, чтобы обновить данные для Thingspeak, нам также понадобится **Write API Key** на своем канале.
{% highlight C %}
// You need to edit with your ThingSpeak information.
String writeAPIKey="YOUR_WRITE_API_KEY";
{% endhighlight %}

Запрос **HTTP POST** выполняется здесь, чтобы запросить обновление 2 значений для **field1** и **field2**. В этом коде мы используем тип данных **JSON** для отправки информации. Кроме того, вы можете увидеть строки кода, которые являются комментариями, если вы хотите отправить данные типа **urlencoded**.
{% highlight C %}
Serial.println("Sending POST request:");
// String postData= "api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
String postData = "{\"api_key\":\"" + writeAPIKey + "\",\"field1\":\"" + String(t) + "\",\"field2\":\"" + String(h) + "\"}";
// client.println("POST /update HTTP/1.1");
client.println("POST /update.json HTTP/1.1");
client.println( "Host: api.thingspeak.com" );
client.println( "Connection: close" );
// client.println( "Content-Type: application/x-www-form-urlencoded" );
client.println( "Content-Type: application/json" );
client.println( "Content-Length: " + String( postData.length() ) );
client.println();
client.println( postData );
Serial.println( postData );
{% endhighlight %}

После **загрузки** программы в модуль мы можем посмотреть **код ответа**, возвращаемый через **Serial Monitor**. Код возврата **200 OK** означает, что запрос на обновление данных выполнен успешно (Рис. 4).
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/SerialMonitor3.png" style="zoom:50%">
  <br>
    <em>Рисунок 4. Код ответа от сервера</em>
</p>

**b. Использование ESP8266WiFi.h** и **ESP8266HTTPClient.h**. Мы можем использовать следующий пример кода:
{% highlight C %}
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include "DHT.h"

// You need to edit with your network credentials.
#define WIFI_NAME "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"

// You need to edit with your ThingSpeak information.
String writeAPIKey = "YOUR_WRITE_API_KEY";

// DHT information
#define DHTPIN D2
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastTime = 0;
unsigned long delayTime = 20000; // set a period of sending data.

void setup()
{
  Serial.begin(115200);
  Serial.println();

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.print("Connected, IP address: ");
  Serial.println(WiFi.localIP());

  // Begin DHT Sensor
  dht.begin();
  
  delay(2000);
}

void loop() {
    //Send an HTTP POST request every 20 seconds
    if ((millis() - lastTime) > delayTime) {
        httpPostRequest();
        lastTime = millis();
    }
}

void httpPostRequest() {
    WiFiClient client;
    HTTPClient http;
    
    // Read DHT sensor
    float h = dht.readHumidity();    // Read humidity as %
    float t = dht.readTemperature(); // Read temperature as Celsius (the default)
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t)) {
        Serial.println(F("Failed to read from DHT sensor!"));
        return;
    }
    
    http.begin(client, "http://api.thingspeak.com/update.json");
    /*
    // Specify content-type header
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");
    // Data to send with HTTP POST
    String httpRequestData = "api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
    // Send HTTP POST request
    int httpResponseCode = http.POST(httpRequestData);
    */
    
    // If you need an HTTP request with a content type: application/json, use the following:
    http.addHeader("Content-Type", "application/json");
    // JSON data to send with HTTP POST
    String httpRequestData = "{\"api_key\":\"" + writeAPIKey + "\",\"field1\":\"" + String(t) + "\",\"field2\":\"" + String(h) + "\"}";
    // Send HTTP POST request
    Serial.println("Sending HTTP POST request");
    int httpResponseCode = http.POST(httpRequestData);
     
    String payload = "{}"; 
  
    if (httpResponseCode>0) {
        Serial.print("HTTP Response code: ");
        Serial.println(httpResponseCode);
        payload = http.getString();
        Serial.println(payload);
    }
    else {
        Serial.print("Error code: ");
        Serial.println(httpResponseCode);
    }
    // Free resources
    http.end();
}
{% endhighlight %}

В данном коде, библиотека **ESP8266HTTPClient.h** позволяет нам создавать экземпляр объекта **HTTPClient**, и здесь поддерживаются методы **HTTP-запросов**. Объект **http** вызывает метод **POST()** для отправки запроса **POST URL**.
{% highlight C %}
WiFiClient client;
HTTPClient http;
http.begin(client, "http://api.thingspeak.com/update.json");
/*
// Specify content-type header
http.addHeader("Content-Type", "application/x-www-form-urlencoded");
// Data to send with HTTP POST
String httpRequestData = "api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
// Send HTTP POST request
int httpResponseCode = http.POST(httpRequestData);
*/
    
// If you need an HTTP request with a content type: application/json, use the following:
http.addHeader("Content-Type", "application/json");
// JSON data to send with HTTP POST
String httpRequestData = "{\"api_key\":\"" + writeAPIKey + "\",\"field1\":\"" + String(t) + "\",\"field2\":\"" + String(h) + "\"}";
// Send HTTP POST request
Serial.println("Sending HTTP POST request");
int httpResponseCode = http.POST(httpRequestData);
{% endhighlight %}

После отправки запроса **HTTP POST** сервер отправляет обратный ответ с **HTTP-кодом**, чтобы подтвердить, что запрос успешно отправлен. Кроме того, Thingspeak отправляет информацию о **канале** в формате **JSON**.

{% highlight C %}
String payload = "{}"; 
  
if (httpResponseCode>0) {
    Serial.print("HTTP Response code: ");
    Serial.println(httpResponseCode);
    payload = http.getString();
    Serial.println(payload);
}
else {
    Serial.print("Error code: ");
    Serial.println(httpResponseCode);
}
{% endhighlight %}

Как и в приведенном выше случае, после **Загрузки** программы в модуль мы можем посмотреть **Код ответа** на **Serial Monitor**. Код возврата **200 OK** означает, что запрос на обновление данных был выполнен успешно (Рис. 5).
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/SerialMonitor4.png" style="zoom:50%">
  <br>
    <em>Рисунок 5. Код ответа от сервера</em>
</p>

### 4. Заключение <a name="Section4"></a>
Таким образом, мы узнали как использовать **HTTP POST-запрос** для обновления данных на Thingspeak. При двух способах программирования для NodeMCU мы оба получаем одинаковые результаты, данные успешно обновляются на Thingspeak и возвращают код **200 ОК** (Рис. 6).
<p align="center">
  <img alt="Bieu Do" src="/tech-blog/assets/Thingspeak_BieuDo2.png" style="zoom:50%">
  <br>
    <em>Рисунок 6. Графики отображения полученных данных Thingspeak</em>
</p>

Можно делать вывод о том, что Thingspeak поддерживает множество различных методов отправки данных в канал. Каждый способ подойдет для разных типов устройств.

Опять же, методы GET и POST, основанные на протоколе HTTP, могут занимать много ресурсов на линии передачи данных, если количество устройств Интернета вещей увеличивается нерпрервыно. Поэтому сейчас для Интернета вещей разрабатывается ряд других протоколов, таких как MQTT, CoAP, которые занимают меньше ресурсов для отправки данных датчиков.

### 5. Дополнительные источники <a name="Section5"></a>
- Пример кода: [https://github.com/itcomnetworks/IoT-Series](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}
- [Write Data - Update channel data with HTTP GET or POST](https://www.mathworks.com/help/thingspeak/writedata.html){:target="_blank"}
- [HTTP Request Methods](https://www.w3schools.com/tags/ref_httpmethods.asp){:target="_blank"}
- DHT Sensor Library: [https://github.com/adafruit/DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library){:target="_blank"}