# WeatherBox 项目

作者邮箱：luhuadong@163.com，更新日期：2022-08-26

![](./images/WeatherBox-Cloud-Show-09.jpeg)

## 项目介绍

本项目使用 Wio Terminal 在天气小助手的基础上，实现一个可远程访问的空气质量监测系统。我们将利用 Wio Terminal 提供的 LCD 显示屏、WiFi 无线模块以及 Grove 扩展接口连接传感器，实现实时监测室内环境温湿度、PM2.5、AQI 空气质量指数等数据。不但可以通过 WiFi 无线网络获取当地天气信息以及未来三天的天气预报，使用按键即可切换显示界面查看详细信息。还可以将采集的空气质量数据通过 MQTT 协议上报到阿里云物联网平台，并且通过 IoT Studio 创建出可随时随地访问的 Web 界面。

本项目的网络通信链路很长，不仅涉及 I2C 总线协议这样的芯片级/板级通信协议，还涉及 HTTP/HTTPS、MQTT 这些广泛应用的网络通信协议。横跨设备端嵌入式开发、云端开发、Web 端开发等三大板块，相信通过本项目的学习和实践，读者朋友们一定会学到很多知识！

### 主要功能

- 开机自动联网获取实况天气和预报天气
- 在主界面同时显示室外和室内的温湿度
- 增加了 PM2.5 和 AQI（空气质量指数）
- 按上方左键可手动更新天气信息
- 可通过五向开关的 Left 和 Right 键翻页查看未来几天的天气预测
- 将传感器采集的数据实时上报到物联网云平台
- 通过 Web 界面远程访问传感器的数据

### 学习重点

- LCD 屏幕内容显示和刷新
- 通过 I2C 采集传感器数据（I2C 总线挂载多个从设备）
- 通过 ADC 采集模拟量数据
- WiFi 无线网络的连接和使用
- 通过 Web API 获取天气信息
- 解析 JSON 格式数据
- 消除按键抖动的方法
- 通过 MQTT 协议连接阿里云物联网平台
- 通过 Web 网页展示传感器数据

### 材料清单

**硬件**

- 1 x [Wio Terminal](https://wiki.seeedstudio.com/Wio-Terminal-Getting-Started/)（含 USB Type-C 数据线）
- 1 x [DHT20 温湿度传感器](https://wiki.seeedstudio.com/Grove-Temperature-Humidity-Sensor-DH20/)（含 Grove 连接线）
- 1 x [HM3301 激光式 PM2.5 传感器](https://wiki.seeedstudio.com/Grove-Laser_PM2.5_Sensor-HM3301/)（含 Grove 连接线）
- 1 x [AQI 空气质量传感器](https://wiki.seeedstudio.com/Grove-Air_Quality_Sensor_v1.3/)（含 Grove 连接线）
- 1 x [Grove I2C 扩展模块](https://wiki.seeedstudio.com/Grove-I2C_Hub/)（含 Grove 连接线）

**软件**

- Arduino IDE
- 阿里云物联网平台（包括 IoT Studio）



## 整体框架

项目的整体架构和工作原理如下图所示，Wio Terminal 有两类数据源，一类是通过底下的两个 Grove 接口（左侧为 I2C 接口，右侧为 GPIO/ADC/PWM 接口）连接 DHT20 传感器、PM2.5 传感器和 AQI 空气质量指数传感器，共采集环境温度、湿度、PM2.5 和 AQI 四组数据。另一类数据是通过 Wi-Fi 网络从 Web 服务器获取的 JSON 格式的天气数据，我们会将获取到的本地传感器数据，以及从 JSON 解析出来的天气数据展示在 LCD 屏幕上。

![](../images/Wio-Terminal-WeatherBox-Advanced.png)

另外，为了实现更有趣的的物联网应用，我们还将通过 Wi-Fi 无线网络，使用 MQTT 协议将 Wio Terminal 连接到 IoT 平台。为了方便国内用户使用，本项目选择了阿里云物联网平台，将传感器数据按照 IoT 平台的格式要求打包，并上报到 IoT 平台。

最后，我们还使用阿里云 IoT Studio 开发工具，为该项目创建了一个可远程访问的 Web 界面，用于展示 Wio Terminal 传感器上报的数据。用户可以通过电脑或者手机等设备，随时随地查看被监测场所的空气质量信息。



## 准备工作

### 支持开发板

为了可以使用 Arduino IDE 开发 Wio Terminal 程序，首先需要安装相应的开发板支持包，Wio Terminal 对应的是 Seeed SAMD Boards。

打开 Arduino IDE，点击 *File（文件） > Preference（偏好设置）* ，打开“首选项”页面，将以下网址复制到“附加开发板管理器网址”（Additional Boards Manager URLs）一栏。

```bash
https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
```

然后点击 *Tools（工具） > Board（开发板）> Boards Manager…* ，打开“开发板管理器”，在搜索栏中搜索关键字 **Wio Terminal** 后，点击并安装 **Seeed SAMD Boards** 最新版本。

![](../images/Arduino-IDE-Board-Manager-add-Seeed-SAMD.png)

在 *Tools（工具）> Board（开发板）* 菜单中选择“Seeed SAMD”项，选择 **Seeed Wio Terminal** 开发板。

从 *Tools（工具）> Serial Port（端口）* 中选择 Wio Terminal 的串行设备。在 Ubuntu 系统中通常为 /dev/ttyACM0，Windows 系统则是 COM 端口。如果你不知道具体是哪个，可以先断开 Wio Terminal 并重新打开菜单，消失的条目应该是它的串口，接着重新连接电路板并选择该串行端口即可。



### 安装依赖库

本项目依赖 **LCD** 库、**rpcWiFi** 库、**ArduinoJson** 库、**DHT20** 库、**Seeed_PM2_5_sensor_HM3301** 库、**Grove_Air_quality_Sensor** 库、**Bounce2** 库和 **pubsubclient** 库：

- `LCD` 库在安装 *Seeed SAMD Boards* 库时已经包含了；
- `rpcWiFi` 库可以在 [GitHub 仓库](https://github.com/Seeed-Studio/Seeed_Arduino_rpcWiFi) 下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库；
- `ArduinoJson` 库可以在 [GitHub 仓库](https://github.com/bblanchon/ArduinoJson)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- `DHT20` 库可以在 [GitHub 仓库](https://github.com/RobTillaart/DHT20)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- `Seeed_PM2_5_sensor_HM3301` 库可以在 [GitHub 仓库](https://github.com/Seeed-Studio/Seeed_PM2_5_sensor_HM3301)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- `Grove_Air_quality_Sensor` 库可以在 [GitHub 仓库](https://github.com/Seeed-Studio/Grove_Air_quality_Sensor)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- `Bounce2` 库可以在 [GitHub 仓库](https://github.com/thomasfredericks/Bounce2)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- `PubSubClient` 库可以在 [GitHub 仓库](https://github.com/knolleary/pubsubclient)下载，在 Arduino IDE 点击 *项目 > 加载库 > 添加 .ZIP 库…* 即可添加库。
- 另外，本项目还使用了 `Free_Fonts.h` 库提供的一些免费字体，可以点击[这里](https://files.seeedstudio.com/wiki/Wio-Terminal/res/Free_Fonts.h)下载，并将它放在 Arduino 工程中。

温馨提示：

- 上面提到的这些库也可以在 Arduino IDE 库管理器中搜索并安装。
- 如果无线网卡操作失败，请更新 RTL8720 固件后再次尝试，更新步骤参考[这里](https://wiki.seeedstudio.com/Wio-Terminal-Network-Overview/)。



### 物联网平台

在开始编写代码之前，请先登录 [阿里云物联网平台](https://iot.aliyun.com/)（如无账户请先创建账户），进入控制台，开通公共实例。

![](../images/Aliyun-IoT-Console.png)

然后依次创建产品（WeatherBox 天气小助手）和设备（Wio Terminal）。其中，产品的连网协议选择 Wi-Fi，数据格式选择 ICA 标准数据格式（Alink JSON）。

![](../images/Aliyun-IoT-WeatherBox-01.png)

下一步我们需要为产品设置具体的功能定义，这里定义了四个属性，分别对应 Wio Terminal 上报的四组传感器数据类型。即温度（Temperature）、湿度（Humidity）、PM2.5（PM25）和 空气质量指数（AQI），括号中的是属性的标识符，这个标识符在使用 MQTT 上报时需要用到，因此需要特别注意保持两端的标识符一致。另外，属性还需要设置数据类型和数据范围等参数，具体如下图所示。

![](../images/Aliyun-IoT-WeatherBox-02.png)

切换到“设备”页面，为“天气小助手”产品添加一个设备 wiot_01。设备创建成功后，点击“查看”进入设备详情页面。阿里云物联网平台通过设备三元组信息识别不同产品的不同设备，三元组信息包括 ProductKey、DeviceName 和 DeviceSecret，点击右上方的“查看”可以看到三元组信息，你需要保存下来以便后续填写到代码当中。由于本项目没有使用阿里云物联网平台提供的设备端 SDK，而是直接通过 MQTT 协议连接云平台，因此你还需要点击左下方的“查看”获取完整的 MQTT 连接参数，包括 clientId、username、passwd、mqttHostUrl 和 port 信息。

![](../images/Aliyun-IoT-WeatherBox-03.png)

至此，阿里云物联网云平台的准备工作已经完成。



## 代码实现

### 创建工程

打开 Arduino IDE，点击“文件 -> 新建”，按 Ctrl + S 将文件保存为 WeatherBox-Cloud 项目。

![](../images/ArduinoIDE-New-WeatherBox-Cloud.png)



### LCD 显示

本项目使用 Wio Terminal 开发板软件包提供的 TFT_eSPI 库来绘制屏幕，首先创建一个 tft 对象，创建完成后进行初始化，之后便可以调用 TFT_eSPI 类提供的 `fillScreen()`、`setFreeFont()` 等方法进行绘制或设置。

```cpp
#include "TFT_eSPI.h"
TFT_eSPI tft;

void setup() {
    tft.begin();
    tft.setRotation(3);
    tft.fillScreen(tft.color565(24,15,60));
    tft.fillScreen(TFT_NAVY);
    tft.setFreeFont(FMB12);
    tft.setCursor((320 - tft.textWidth("Seeed Weather Box"))/2, 100);
    tft.print("Seeed Weather Box");
    ...
}
```




### 读取传感器

读取 DHT20 传感器温湿度数据的示例代码如下，这里封装了一个 `updateSensorData()` 函数直接将读取到的温湿度数据显示到 LCD 屏幕上。

```cpp
#include "DHT20.h"

DHT20     DHT;

void setup() {
    Serial.begin(115200);

    if (! DHT.begin()) {
        Serial.println("Could not find AHT Sensor? Check wiring");
        while (1) delay(10);
    }
    Serial.println("DHT20 init OK!");
}

void updateSensorData()
{
    int status = DHT.read();
    switch (status)
    {
    case DHT20_OK:
      Serial.print("OK,\t");
      break;
    case DHT20_ERROR_CHECKSUM:
      Serial.print("Checksum error,\t");
      break;
    case DHT20_ERROR_CONNECT:
      Serial.print("Connect error,\t");
      break;
    case DHT20_MISSING_BYTES:
      Serial.print("Missing bytes,\t");
      break;
    default:
      Serial.print("Unknown error,\t");
      break;
    }

    drawTempValue(DHT.getTemperature());
    drawHumiValue(DHT.getHumidity());
}
```

另外，本项目在 WeatherBox 基础版的基础上增加了 PM2.5 和 AQI 传感器，具体用法可在完整代码中查看。



### 抓取天气数据

rpcWiFi 库提供了 **HTTPClient**，我们可以通过它来发送 HTTP GET、POST 或者 PUT 请求到 Web 服务器，并接收服务器返回的数据。我们可以找到很多提供天气信息的 Web API，例如本项目使用的[高德地图 API](https://lbs.amap.com/api/webservice/guide/api/weatherinfo)，可以获取全国各城市的实况天气及天气预测。其中，GET 请求的 URL 如下：

**实时天气**（当天）

```bash
https://restapi.amap.com/v3/weather/weatherInfo?city=440100&key=yourkey
```

**天气预测**（未来三天）

```bash
https://restapi.amap.com/v3/weather/weatherInfo?city=440100&key=yourkey&extensions=all
```

参数说明：

- `city` 是城市编码，比如 440100 代表广州；（[城市编码表](https://lbs.amap.com/api/webservice/download)）
- `key` 是应用对应的代码，需要在平台申请（提示：将 `yourkey` 替换为你申请的 Key 代码）；
- `extensions` 表示获取类型，缺省值是 `base`，表示获取实况天气，`all` 表示获取预报天气；
- `output` 表示返回格式，可选 JSON 或 XML，默认返回 JSON 格式数据。

以实时天气 API 为例，返回的 JSON 数据如下：

```json
{
    "status":"1",
    "count":"1",
    "info":"OK",
    "infocode":"10000",
    "lives":[
        {
            "province":"广东",
            "city":"广州市",
            "adcode":"440100",
            "weather":"多云",
            "temperature":"27",
            "winddirection":"南",
            "windpower":"≤3",
            "humidity":"90",
            "reporttime":"2022-06-22 23:31:23"
        }
    ]
}
```



### 解析 JSON 数据

有了 JSON 数据之后，我们还需要将它解析出来，这里使用 ArduinoJson 库进行操作。为了将解析出来的数据保存起来，我们定义了 `lives_t` 和 `forecasts_t` 结构体数据类型，分别对应实况天气和预测天气数据。

```cpp
typedef struct lives {
    char province[16];
    char city[16];
    char adcode[16];
    char weather[16];
    char temperature[16];
    char humidity[16];
    char winddirection[16];
    char windpower[16];
    char reporttime[16];
} lives_t;

lives_t lives_data;

typedef struct forecasts {
    char date[16];
    char week[16];
    char dayweather[16];
    char nightweather[16];
    char daytemp[16];
    char nighttemp[16];
    char daywind[16];
    char nightwind[16];
    char daypower[16];
    char nightpower[16];
} forecasts_t;

#define FORECASTS_SIZE  4
forecasts_t forecasts_data[FORECASTS_SIZE];
```

JSON 解析的部分代码如下：

```cpp
DynamicJsonDocument doc(capacity);
deserializeJson(doc, payload);

strncpy(lives_data.province, doc["lives"][0]["province"], STR_SIZE_MAX);
strncpy(lives_data.city, doc["lives"][0]["city"], STR_SIZE_MAX);
strncpy(lives_data.weather, doc["lives"][0]["weather"], STR_SIZE_MAX);
strncpy(lives_data.temperature, doc["lives"][0]["temperature"], STR_SIZE_MAX);
strncpy(lives_data.humidity, doc["lives"][0]["humidity"], STR_SIZE_MAX);
strncpy(lives_data.winddirection, doc["lives"][0]["winddirection"], STR_SIZE_MAX);
strncpy(lives_data.windpower, doc["lives"][0]["windpower"], STR_SIZE_MAX);
strncpy(lives_data.reporttime, doc["lives"][0]["reporttime"], STR_SIZE_MAX);
```



### MQTT 连接 IoT 平台

我们通过 PubSubClient 库完成 MQTT 的连接、发布和订阅功能，首先需要引入头文件。

```cpp
#include <PubSubClient.h>
```

然后将前面在阿里云物联网平台创建设备获得的三元组信息和 MQTT 连接参数，填写到宏定义中。另外，还需要定义一个用于上报属性的主题 `PROPERTY_TOPIC`，这个主题的格式是阿里云 IoT 平台规定。

```cpp
#define ProductKey     "a1eubzkCwuz"
#define DeviceName     "wiot_01"
#define DeviceSecret   "56e89exxxxxxxxxxxxxxxxxxxxf362b4"

#define MQTT_HOST      "a1eubzkCwuz.iot-as-mqtt.cn-shanghai.aliyuncs.com"
#define CLIENTID       "a1eubzkCwuz.wiot_01|securemode=2,signmethod=hmacsha256,timestamp=1657691211776|"
#define USERNAME       "wiot_01&a1eubzkCwuz"
#define PASSWORD       "13ef429e9cxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxdfaf05f13"
#define MQTT_PORT      1883

#define PROPERTY_TOPIC "/sys/"ProductKey"/"DeviceName"/thing/event/property/post"
```

创建 MQTT 客户端

```cpp
WiFiClientSecure wifiClient;
PubSubClient mqttClient(wifiClient);
```

在 setup 函数中设置 MQTT 服务器地址、保活参数、消息回调函数等必要参数，然后就可以连接云平台了。需要特别注意的是，阿里云 IoT 平台要求的 MQTT 连接保活时间的取值范围为 30 秒 ~ 1200 秒，如果设置的 KeepAlive 值不在此范围内将导致连接失败，建议取值 300 秒以上。

```cpp
void setup()
{
    mqttClient.setServer(MQTT_HOST, MQTT_PORT); // Connect the MQTT Server
    mqttClient.setKeepAlive(300); // Must be in 30s - 1200s
    mqttClient.setCallback(mqtt_callback);
    
    if (!mqttClient.connected()) {
        mqtt_connect();
    }
}
```

连接时传入 CLIENTID、USERNAME 和 PASSWORD 参数，如下：

```cpp
void mqtt_connect()
{
  // Loop until we're reconnected
  while (!mqttClient.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    // Attempt to connect
    if (mqttClient.connect(CLIENTID, USERNAME, PASSWORD)) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}
```



### 上报传感器数据

MQTT 连接成功后，就可以周期性地将采集的传感器数据上报到阿里云 IoT 平台。为了简单操作，这里直接使用字符串来组合 Alink JSON 数据格式。定义了一个 `UPLOAD_MESSAGE` 宏，发布时只需要将数据填入对应的位置即可。

```cpp
#define UPLOAD_MESSAGE "{\"id\":\"123\",\"version\":\"1.0\",\"sys\":{\"ack\":0},\"params\":{\"Temperature\":{\"value\":%.1f},\"Humidity\":{\"value\":%.1f},\"PM25\":{\"value\":%u},\"AQI\":{\"value\":%u}},\"method\":\"thing.event.property.post\"}"
```

消息发布

```cpp
char msg[512];
snprintf(msg, sizeof(msg), UPLOAD_MESSAGE, Temperature, Humidity, PM25, AQI);
mqttClient.publish(PROPERTY_TOPIC, msg);
```



### 完整代码

完整代码可从附件中下载。需要注意的是，在编译之前，请修改程序开头的字符串常量 `ssid` 和 `password` 替换成你的 WiFi 网络；将 `URL_BASE` 和 `URL_ALL` 中的 `cityCode` 替换成需要查询的城市，将 `yourKey` 替换成你的 Key。

```cpp
const char *ssid     = "yourNetwork";
const char *password = "yourPassword";
const char *URL_BASE = "https://restapi.amap.com/v3/weather/weatherInfo?city=cityCode&key=yourKey";
const char *URL_ALL  = "https://restapi.amap.com/v3/weather/weatherInfo?city=cityCode&key=yourKey&extensions=all";
```



## 创建 Web 可视化界面

打开阿里云 IoT Studio（https://studio.iot.aliyun.com），首先在左侧栏选择“项目管理 -> 普通项目”，点击“新建项目”创建项目，这里取名为 WeatherBox。然后在“应用开发 -> Web应用”页面，点击“新建”按钮创建一个 Web 应用，这里取名为 WeatherBox_Web。

![](../images/Aliyun-IoT-Studio-New.png)

在“产品”选项卡中，关联我们前面在阿里云物联网平台创建的产品 WeatherBox 天气小助手。这样就可以读取该产品的设备数据，并将其绑定到 Web 前端的对应组件上。

![](../images/Aliyun-IoT-Studio-Product.png)

进入 Web 可视化开发界面，为 WeatherBox_Web 应用添加元素、绑定数据。一个简单的设计如下图所示。

![](../images/Aliyun-IoT-Studio-Web-Full.png)

设计完成后，点击右上角的“发布”按钮即可发布 WeatherBox_Web 应用。发布成功后，你将获得一个可公网访问的 URL 链接，例如：https://a120xa0fxhfmviea.vapp.cloudhost.link/page/1004307?token=c05631b8aa553c14052dac18f365c329。当然，你也可以将 Web 应用绑定到你自己的域名，这样更便于访问。

扫描下方二维码，即可访问 WeatherBox_Web 应用。

![](../images/qrcode.png)



## 运行效果

点击 Arduino IDE 工具栏中的“上传”按钮，编译并上传程序到 Wio Terminal，运行效果如下图所示。

【主界面】同时显示室内外的温湿度数据、AQI 空气质量指数，以及 PM2.5 的值

![](../images/WeatherBox-Cloud-Show-08.jpeg)

【详情页面】按方向键（五向开关的左右键）切换到天气预报界面，可查看未来三天的数据

![](../images/WeatherBox-Cloud-Show-03.jpeg)

【Web 端】远程访问 Web 界面，随时随地了解空气质量情况

![](../images/WeatherBox-Cloud-Show-06.jpeg)



## 参考

- [MQTT-TCP连接通信](https://help.aliyun.com/document_detail/73742.htm)
- [Paho-MQTT C接入示例](https://help.aliyun.com/document_detail/146611.html)
- [在支持MQTT的模组上集成SDK](https://help.aliyun.com/document_detail/111903.html)
- [Set up Wio Terminal as MQTT Display for machinechat JEDI One Sensor Data](https://forum.digikey.com/t/set-up-wio-terminal-as-mqtt-display-for-machinechat-jedi-one-sensor-data/16532)



```cpp
#if 0
const char *ssid = "ASENSING-GUEST";
const char *password = "88888888";
#else
const char *ssid = "FCTC_89";
const char *password = "Lu15899962740";
#endif

// 440100 441802
const char* URL_BASE = "https://restapi.amap.com/v3/weather/weatherInfo?city=440100&key=ac901c195798b1f2767987a55ee74156";
const char* URL_ALL  = "https://restapi.amap.com/v3/weather/weatherInfo?city=440100&key=ac901c195798b1f2767987a55ee74156&extensions=all";

#define ProductKey     "a1eubzkCwuz"
#define DeviceName     "wiot_01"
#define DeviceSecret   "56e89e401e0ed9b10e1e0e5725f362b4"

#define MQTT_HOST      "a1eubzkCwuz.iot-as-mqtt.cn-shanghai.aliyuncs.com"
#define CLIENTID       "a1eubzkCwuz.wiot_01|securemode=2,signmethod=hmacsha256,timestamp=1657691211776|"
#define USERNAME       "wiot_01&a1eubzkCwuz"
#define PASSWORD       "13ef429e9c6206d45ceefd3d73e4385ce5ac52de7d7c9b7c12b5d3fdfaf05f13"
#define MQTT_PORT      1883
```

