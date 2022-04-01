---
layout: post
title:  "IoT-Ep4 | HTTP GET отправляет данные в Thingspeak"
date:   2022-03-19 00:00:00 +0300
category: Tech
image: /tech-blog/assets/GETRequest.png
tags: [IoT, Thingspeak, HTTP]
lang: ru
lang-ref: HTTP GET sends data to Thingspeak
---
В этой статье мы рассмотрим, как использовать метод GET (запрос HTTP GET) для обновления данных датчиков на Thingspeak.
Вы можете посмотреть демо-видео или инструкцию ниже.

<p align="center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/key3GB2gg70" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

# Содержание
1. [Введение](#Section1)
2. [Запросы API GET](#Section2)
3. [Соединение и программирование](#Section3)
- [3.1. Соединение](#Subsection3.1)
- [3.2. Использование запроса HTTP GET](#Subsection3.2)
4. [Заключение](#Section4)
5. [Дополнительные запросы](#Section5)

### 1. Введение <a name="Section1"></a>
В предыдущих двух статьях мы узнали о Thingspeak и о том, как отправлять данные температуры и влажности с датчика DHT22 в Thingspeak с помощью библиотеки Thingspeak, разработанной MathWorks. Кроме того, платформа Thingspeak также предоставляет API (Application Programming Interface) или интерфейсы прикладного программирования, которые позволяют разработчикам приложений писать программы для связи с Thingspeak с любого устройства, поддерживающего сетевой стек TCP/IP. Таким образом, используя эти API, мы можем программировать устройства, отличные от NodeMCU, для отправки данных в Thingspeak.

Протокол HTTP является одним из самых популярных протоколов на сегодняшний день, потому что почти все мы ежедневно используем веб-браузеры, а протокол HTTP играет важную роль в передаче веб-данных в Интернете. **HTTP RESTfull** — это один из веб-сервисов, который использует протокол HTTP для обмена данными с такими запросами, как: GET, PUT, POST, DELETE, что эквивалентно чтению, обновлению, созданию и удалению файлов, операциям, связанным с ресурсами. С помощью запросов **HTTP Request** данные могут быть удалены и обновлены в поле канала на Thingspeak.

### 2. Запросы API GET <a name="Section2"></a>
Чтобы узнать о запросах **API**, поддерживаемых Thingspeak, мы можем найти в вкладке **API Keys** на любой странице канала (Рис. 1).
<p align="center">
  <img alt="Edit bieu do" src="/tech-blog/assets/APIRequests.png" style="zoom:50%">
  <br>
    <em>Рисунок 1. API</em>
</p>

- **Пример 1**: Использование **запроса GET** для записи данных в поле.
{% highlight http %}
GET https://api.thingspeak.com/update?api_key=YOUR_WRITE_API_KEY&field1=0
{% endhighlight %}
Запрос здесь построен с помощью **https://api.thingspeak.com/update?api_key=** + **OurWriteAPIKey** + **&** + **Fieldname** + **=** + **значение**. С помощью **запрос GET** мы можем вставить ссылку в веб-браузер (Рис. 2). Если **Запрос** выполнен успешно, на веб-странице будет возвращено число. Затем значение 40 будет записано в поле номер 1 **Field1** канала.
<p align="center">
  <img alt="API requests" src="/tech-blog/assets/APIRequests2.png" style="zoom:50%">
  <br>
    <em>Рисунок 2. Проверка запроса GET</em>
</p>

Кроме того, чтобы записать данные во множество разных полей, мы можем добавить **&field2=30**, например, следующим образом:
{% highlight http %}
GET https://api.thingspeak.com/update?api_key=YOUR_WRITE_API_KEY&field1=10&field2=30
{% endhighlight %}

Метод HTTP GET можно изобразить на Рис. 3.
<p align="center">
  <img alt="HTTP GET" src="/tech-blog/assets/HTTPGET2.png" style="zoom:50%">
  <br>
    <em>Рисунок 3. Метод HTTP GET</em>
</p>

Согласно Рис. 3, мы можем написать запрос HTTP GET в следующем виде:

|GET /update?api_key=RBDRQF9W9SDPJA6T&field1=10&field2=30 |
|Host: api.thingspeak.com                                 |

Для программирования на Arduino IDE, мы используем объект **client** из **WiFiClient** в соответствии с:
{% highlight C++ %}
String getUrl = "/update?api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
// Make a HTTP request:
client.println("GET " + getUrl + " HTTP/1.1");
client.println("Host: api.thingspeak.com");
client.println("Connection: close");
client.println();
{% endhighlight %}

- **Пример 2**: Использование запроса HTTP GET для чтения данных канала
{% highlight http %}
GET https://api.thingspeak.com/channels/1679735/feeds.json?api_key=56R00C2TUIT40NY6&results=2
{% endhighlight %}
Выполняя этот запрос в веб-браузере, возвращается строка **json**, отображающая информацию о канале и значения полей.
{% highlight json %}
{"channel":{"id":1679735,"name":"Hệ thống quan trắc thời tiết",
"description":"Hệ thống theo dõi thời tiết để thu thập dữ liệu nhiệt độ và độ ẩm từ xa.","latitude":"0.0","longitude":"0.0",
"field1":"Nhiệt độ","field2":"Độ ẩm","created_at":"2022-03-19T12:14:27Z","updated_at":"2022-03-19T12:14:28Z","last_entry_id":1259},
"feeds":[{"created_at":"2022-03-19T21:49:49Z","entry_id":1258,"field1":"27.10000","field2":"11.70000"},
{"created_at":"2022-03-19T21:50:14Z","entry_id":1259,"field1":"27.10000","field2":"11.30000"}]}
{% endhighlight %}

### 3. Соединение и программирование <a name="Section3"></a>
#### 3.1. Соединение <a name="Subsection3.1"></a>
В этой статье мы продолжаем работать с NodeMCU и DHT22. Датчик можно подключить к модулю NodeMCU по схеме на Рис. 4.
<p align="center">
  <img alt="Library Manager" src="/tech-blog/assets/NodeMCU_DHT.png" style="zoom:50%">
  <br>
    <em>Рисунок 4. Схема соединение NodeMCU с DHT22</em>
</p>

#### 3.2. Использование запроса **HTTP GET** <a name="Subsection3.2"></a>
В этом примере нам нужно обновить 2 поля: **field1** для температуры и **field2** для влажности. Вы можете найти пример скетча [здесь](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}.

Существует 2 способа программирования в Arduino IDE для NodeMCU для отправки **запроса HTTP GET** на удаленный сервер:
- Метод 1: мы используем только 1 библиотеку **ESP8266WiFi.h**.
- Метод 2: мы можем использовать комбинацию двух библиотек **ESP8266WiFi.h** и **ESP8266HTTPClient.h**.

**a. Использование ESP8266WiFi.h**: вы можете скопировать **Sketch** ниже.
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
    HTTPGet();
    lastTime = millis();
  }
}

void HTTPGet() {
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

      // Use HTTP GET request
      Serial.print("Requesting URL: ");
      // Build the GET url: http://api.thingspeak.com/update?api_key=RBDRQF9W9SDPJA6T&field1=0
      String getUrl = "/update?api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
      Serial.println(getUrl);

      // Make a HTTP request:
      client.println("GET " + getUrl + " HTTP/1.1");
      client.println("Host: api.thingspeak.com");
      client.println("Connection: close");
      client.println();
      
    }

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
{% endhighlight %}

Измение информации о подключение модуля к WiFi:
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

Запрос **HTTP GET** выполняется здесь, чтобы запросить обновление 2 значений для **field1** и **field2**.
{% highlight C %}
// Use HTTP GET request
Serial.print("Requesting URL: ");
// Build the GET url: http://api.thingspeak.com/update?api_key=RBDRQF9W9SDPJA6T&field1=0
String getUrl = "/update?api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
Serial.println(getUrl);

// Make a HTTP request:
client.println("GET " + getUrl + " HTTP/1.1");
client.println("Host: api.thingspeak.com");
client.println("Connection: close");
client.println();
{% endhighlight %}

После **загрузки** программы в модуль мы можем отслеживать **Response Code**, возвращаемый через **Serial Monitor**. Код возврата **200 OK** означает, что запрос на обновление данных выполнен успешно (Рис. 5).
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/SerialMonitor1.png" style="zoom:50%">
  <br>
    <em>Рисунок 5. Код ответа на Serial Montior</em>
</p>

**b. Использование ESP8266WiFi.h** и **ESP8266HTTPClient.h**. Вы можете скопировать следующий скетч:
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
        httpGETRequest();
        lastTime = millis();
    }
}

void httpGETRequest() {
    WiFiClient client;
    HTTPClient http;
    
    // Read DHT sensor
    float h = dht.readHumidity();    // Read humidity in %
    float t = dht.readTemperature(); // Read temperature as Celsius (the default)
    // Check if any reads failed and exit early (to try again).
    if (isnan(h) || isnan(t)) {
        Serial.println(F("Failed to read from DHT sensor!"));
        return;
    }

    // Build URL GET path.
    String urlPath = "http://api.thingspeak.com/update?api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);

    http.begin(client, urlPath);
    
    // Send HTTP GET request
    Serial.println("Sending HTTP GET request");
    int httpResponseCode = http.GET();
  
    if (httpResponseCode>0) {
        Serial.print("HTTP Response code: ");
        Serial.println(httpResponseCode);
    }
    else {
        Serial.print("Error code: ");
        Serial.println(httpResponseCode);
    }
    // Free resources
    http.end();
}
{% endhighlight %}

В этом скетче библиотека **ESP8266HTTPClient.h** позволяет нам создавать объект **HTTPClient**, и здесь поддерживаются **HTTP-запросов**. Объект **http** вызывает метод **GET()** для отправки запроса **GET URL**.
{% highlight C %}
WiFiClient client;
HTTPClient http;
    
// Build URL GET path.
String urlPath = "http://api.thingspeak.com/update?api_key=" + writeAPIKey + "&field1=" + String(t) + "&field2=" + String(h);
http.begin(client, urlPath);

// Send HTTP GET request
Serial.println("Sending HTTP GET request");
int httpResponseCode = http.GET();
{% endhighlight %}

После отправки запроса **GET** сервер отправляет обратно **HTTP-код**, чтобы подтвердить, что запрос был успешно отправлен.
{% highlight C %}
if (httpResponseCode>0) {
  Serial.print("HTTP Response code: ");
  Serial.println(httpResponseCode);
}
else {
  Serial.print("Error code: ");
  Serial.println(httpResponseCode);
}
{% endhighlight %}

Как и в приведенном выше случае, после **Загрузки** программы в модуль мы можем посмотреть **Код ответа**, возвращаемый через **Serial Monitor**. Код возврата **200 OK** означает, что запрос на обновление данных был выполнен успешно (Рис. 6).
<p align="center">
  <img alt="Serial Monitor" src="/tech-blog/assets/SerialMonitor2.png" style="zoom:50%">
  <br>
    <em>Рисунок 6. Код ответа</em>
</p>

### 4. Заключение <a name="Section4"></a>
Таким образом, мы можем использовать запрос **HTTP GET** для обновления данных на Thingspeak. При использовании двух способах программирования для NodeMCU мы получаем одинаковые результаты, данные успешно обновляются на Thingspeak и возвращают код **200 ОК** (Рис. 7).
<p align="center">
  <img alt="Bieu Do" src="/tech-blog/assets/Thingspeak_BieuDo2.png" style="zoom:50%">
  <br>
    <em>Рисунок 7. Графики отображения данных на Thingspeak</em>
</p>

Обычно метод **HTTP GET** используется для запроса на получение данных с сервера (Server). Однако, используя **GET URL**, мы также можем отправить запрос с необходимой информацией, чтобы попросить сервер обновить данные.

Однако данные, возвращаемые сервером после запроса GET, может быть большим и ненужным для некоторых устройств Интернета вещей, которые должны быть более энергоэффективными. Поэтому сейчас разрабатывается ряд других протоколов, таких как MQTT, CoAP для обмена данными между устройствами ИВ. 

### 5. Дополнительные источники <a name="Section5"></a>
- Пример скетча: [https://github.com/itcomnetworks/IoT-Series](https://github.com/itcomnetworks/IoT-Series){:target="_blank"}
- [Write Data - Update channel data with HTTP GET or POST](https://www.mathworks.com/help/thingspeak/writedata.html){:target="_blank"}
- [HTTP Request Methods](https://www.w3schools.com/tags/ref_httpmethods.asp){:target="_blank"}
- DHT Sensor Library: [https://github.com/adafruit/DHT-sensor-library](https://github.com/adafruit/DHT-sensor-library){:target="_blank"}
