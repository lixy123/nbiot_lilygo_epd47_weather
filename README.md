nb-iot显示天气的墨水屏

<b>一.功能：</b><br/>
1.本系统由两套设备组成<br/>
2.第1套设备: esp32+sim7020c+DS3231时钟模块<br/>
3.第2套设备: lilygo-epd47 墨水屏+hc08蓝牙硬件<br/>
4.设计成2套设备是为了让墨水屏减少充电频率，电池充满电能支持2-3个月<br/>
5.技术原理:<br/>
A.第1套设备具有休眠功能，在代码设定的时间内定时唤醒，通过sim7020c连接互联网获取天气。通过蓝牙将天气数据传给墨水屏，进入休眠<br/>
B.第2套设备平时休眠，当收到第1套设备的数据后被唤醒，接收天气信息并显示，进入休眠<br/>
<br/>
<b>技术指标：</b><br/>
第1套设备在休眠时电流约12ma, 如每天发送天气一次，1-2分钟处于活动状态，此时电流约为80ma。 普通18650电池一般是2500mah, 估算支撑约8天。建议直接接供电器。<br/>
第2套设备在休眠时电流约0.4ma,如每天刷新天气一次，1分钟处于活动状态，此时电流约为80ma。普通18650电池一般是2500mah, 估算支撑约2-3个月。<br/>

注：<br/>
本应用获取天气仅用nbiot，没采用传统的wifi,原因:<br/>
1.便于放在没有wifi的偏僻处。<br/>
2.有些场所虽然有wifi，但用单片机连接wifi限制，麻烦,甚至根本不能连接。<br/>

<b>二.硬件需求：</b><br/>
1.第一套设备 ESP32 +sim7020c+DS3231
 <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/raw/main/sim7020-1.jpg?raw=true' /> <br/>

  功能：获取天气信息，通过蓝牙将天所信息传给墨水屏，休眠<br/>
  硬件清单：<br/>
  A.esp32<br/>
  最好用带psram的esp32,且编译时打开param:enabled,否则json解析天气会不稳定<br/>
  
  B.sim7020c<br/>
  ESP32  ==> Sim7020c <br/> 
  5V    5v <br/>
  GND   GND <br/>
  12    TX <br/>
  13    RX <br/>
  15    RESET (拉低关闭/开启sim7020 休眠节能) <br/>
  
  C.DS3231<br/>
  ESP32  ==>DS3231<br/> 
  5V    5v <br/>
  GND   GND <br/>
  21    SDA <br/>
  22    SCL <br/>  
  
2.第2套设备 lilygo-epd47 + hc08
  <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/blob/main/ink_weather.jpg?raw=true' /> <br/>
  <img src= 'https://github.com/lixy123/LilyGo-EPD47-HC08/raw/main/ink_chixi.jpg?raw=true' /> <br/>
  功能：显示天气
  A.lilygo-epd47
     主控芯片为esp32，驱动墨水屏显示
     
  B.hc08     
     hc-08是一块 BLE4.0蓝牙模块 (购买时要告诉卖方要双晶振版本，否则不支持一级节能模式)<br/>
     hc-08需要配置成客户模式，一级节能模式,蓝牙名称用AT指令修改为edp47_ink，能防止被别的设备误连<br/>    
     引脚连接:<br/>
     lilygo-epd47 ==> hc-08<br/>
       VCC         VCC<br/>
       14          TX<br/>
       15          RX<br/>
       GND         GND<br/>

hc08 AT命令预处理:<br/>
AT+MODE=1 //设置成一级节能模式(必须)<br/>
AT+NAME=INK_047 //修改蓝牙名称，用于客户端查找蓝牙用<br/>
AT+LED=0 //关闭led灯，省电<br/>
注: 也可以通过连接到lilygo-epd47后,自编程序用lilygo-epd47虚拟串口传入AT命令<br/>

 注：lilygo 公司已设计并销售有专用于 lilygo-epd47 的hc-08模块，给hc08套上了外壳，引脚,集成度好, 已进行AT命令预处理.上手即用 <br/>
 如图, 蓝色小模块部分: <br/>
  <img src= 'https://github.com/lixy123/nbiot_lilygo_epd47_weather/blob/main/hc08.JPG?raw=true' /> <br/>
  
<b>三.代码说明:</b> <br/>
  <b>1.epd47_blue_waker_center_nb_iot 获取天气 </b>  <br/>
  烧录到ESP32开发板<br/> 
   1.1 软件: arduino 1.8.13<br/>
   1.2 用到的库文件:<br/>
   arduino-esp32 版本 1.0.6<br/>
   https://github.com/bblanchon/ArduinoJson 版本: 6<br/>
   1.3开发板选择：ESP32 DEV Module <br/>
   编译分区：HUGE APP<br/>
   PSRAM ENABLED(如果有PSRAM)<br/>
   1.4选择端口，点击烧录<br/>

   <b>2.epd47_blue_waker_show_weather 显示天气 </b>   <br/>
  烧录到LilyGo-EPD47墨水屏<br/>
2.1 软件: arduino 1.8.13<br/>
2.2 用到的库文件:<br/>
https://github.com/espressif/arduino-esp32 版本:1.0.6<br/>
https://github.com/Xinyuan-LilyGO/TTGO_TWatch_Library 最新版本, 仅为了用到它定义的开发板<br/>
https://github.com/Xinyuan-LilyGO/LilyGo-EPD47 最新版本<br/>
https://github.com/bblanchon/ArduinoJson 版本: 6<br/>
https://github.com/ivanseidel/LinkedList 最新版本<br/>
2.3开发板选择：TTGO-T-WATCH / PSRAM ENABLED<br/>
2.4选择端口，点击烧录<br/>
注：<br/>
电池供电，能自动休眠.<br/>
  
<b>四.电流实测:</b><br/>
  第2套设备 lilygo-epd47 + hc08：<br/>
  1.休眠： <1ma <br/>
  2.唤醒后: 50-60ma<br/>
  蓝牙模块官方数据：待机电流约6μA ~2.6mA，<br/>
  墨水屏待机电流约0.17ma，<br/>
  因hc08蓝牙电流持续变化，合计总电流估算约在 0.4-0.8ma之间  <br/>
  3.2500ma的电池约能用 2500*0.8/24/30约4月, 实测一般能用2-3月<br/>
  

  
  
  
