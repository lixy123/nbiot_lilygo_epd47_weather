nb-iot显示天气的墨水屏

<b>一.功能：</b><br/>
1.本系统由两套设备组成<br/>
2.第1套设备: esp32+sim7020c+DS3231时钟模块<br/>
3.第2套设备: lilygo-epd47 墨水屏+hc08蓝牙硬件<br/>
4.设计成2套设备是为了让墨水屏减少充电频率，当电池充满电后，能2-3个月内不用惦记着充电。<br/>
5.技术原理:<br/>
A.第1套设备具有休眠功能，在代码设定的时间内定时唤醒，通过sim7020c连接互联网获取天气。通过蓝牙将天气数据传给墨水屏，进入休眠<br/>
B.第2套设备平时休眠，当收到第1套设备的数据后被唤醒，接收天气信息并显示，进入休眠<br/>
<br/>
<b>技术指标：</b><br/>
第1套设备在休眠时电流约12ma, 如每天发送天气一次，1-2分钟处于活动状态，约80ma电流。 普通18650电池,2500mah, 估算支撑约8天左右。建议直接接供电器。<br/>
第2套设备在休眠时电流约0.4ma,如每天刷新天气一次，1分钟处于活动状态，约80ma电流。普通18650电池,2500mah, 估算支撑约2-3个月左右。<br/>


<b>二.硬件需求：</b><br/>
1.第一套设备 ESP32 +sim7020c+DS3231
 <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/raw/main/sim7020-1.jpg?raw=true' /> <br/>

  功能：获取天气信息，通过蓝牙将天所信息传给墨水屏，休眠<br/>
  硬件清单：
  A.esp32
  最好用带psram的esp32,且编译时打开param:enabled,否则json解析天气会不稳定
  
  B.sim7020c
  ESP32 Sim7020c <br/> 
  5V    5v <br/>
  GND   GND <br/>
  12    TX <br/>
  13    RX <br/>
  15    RESET (拉低关闭/开启sim7020 休眠节能) <br/>
  
  C.DS3231
  ESP32 DS3231<br/> 
  5V    5v <br/>
  GND   GND <br/>
  21    SDA <br/>
  22    SCL <br/>  
  
2.第2套设备 lilygo-epd47 + hc08

 <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/raw/main/ink_chixi.jpg?raw=true' /> <br/>
  功能：显示天气
  A.lilygo-epd47
     主控芯片为esp32，驱动墨水屏显示
     
  B.hc08     
     hc-08是一块 BLE4.0蓝牙模块 (购买时要告诉卖方要双晶振版本，否则不支持一级节能模式)<br/>
     hc-08需要配置成客户模式，一级节能模式,蓝牙名称用AT指令修改为edp47_ink，能防止被别的设备误连<br/>
     注：lilygo 公司已有销售专用于 lilygo-epd47 的hc-08模块，直接插上即用，集成度更好. <br/>
     <br/>
     注：官方数据，一级节能模式电流约 6μA ~2.6mA （待机） /1.6mA（工作）<br/>
         相对于全速模式 8.5mA（待机）/9mA（待机） 节能效果明显<br/>
         hc-08模块每日大部分时间应处于在 6μA ~2.6mA （待机）模式,理论电流消耗极低<br/>
     引脚连接:<br/>
     lilygo-epd47  hc-08<br/>
       VCC         VCC<br/>
       14          TX<br/>
       15          RX<br/>
       GND         GND<br/>

<b>三.代码说明:</b> <br/>
  <b>1.epd47_blue_waker_center_nb_iot 获取天气 </b>  
  烧录到ESP32开发板
    <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/blob/main/sim7020-1.jpg?raw=true' /> <br/>
   1.1 软件: arduino 1.8.13
   1.2 用到的库文件:
   arduino-esp32 版本 1.0.6
   1.3开发板选择：ESP32 DEV Module 
   编译分区：HUGE APP
   PSRAM ENABLED(如果有PSRAM)
   1.4选择端口，点击烧录

   <b>2.epd47_blue_waker_show_weather 显示天气 </b>   
  烧录到LilyGo-EPD47墨水屏
2.1 软件: arduino 1.8.13
2.2 用到的库文件:
https://github.com/espressif/arduino-esp32 版本:1.0.6
https://github.com/Xinyuan-LilyGO/TTGO_TWatch_Library 最新版本, 仅为了用到它定义的开发板
https://github.com/Xinyuan-LilyGO/LilyGo-EPD47 最新版本
https://github.com/bblanchon/ArduinoJson 版本: 6
https://github.com/ivanseidel/LinkedList 最新版本
2.3开发板选择：TTGO-T-WATCH / PSRAM ENABLED
2.4选择端口，点击烧录
注：
电池供电，能自动休眠.
  
<b>四.电流实测:</b><br/>
  第2套设备 lilygo-epd47 + hc08：
  1.休眠： <1ma <br/>
  2.唤醒后: 50-60ma<br/>
  蓝牙模块官方数据：待机电流约6μA ~2.6mA，
  墨水屏待机电流约0.17ma，
  因hc08蓝牙电流持续变化，合计总电流估算约在 0.4-0.8ma之间  <br/>
  3.2500ma的电池约能用 2500*0.8/24/30约4月, 实测一般能用2-3月<br/>
  

  
  
  
