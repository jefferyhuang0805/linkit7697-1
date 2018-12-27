#include <LWiFi.h>
#include "MCS.h"
#include "DHT.h"
#include <LiquidCrystal_I2C.h>

DHT dht(4, DHT11);
LiquidCrystal_I2C lcd_i2c(0x27);

int t; //溫度
int h; //濕度
int s; //土壤濕度
int f; //風扇
int w; //噴水
int th = 25; //風扇自動開關溫度預設值
char ss[3]; //土壤濕度文字格式
char _lwifi_ssid[] = "lita";
char _lwifi_pass[] = "0911027768";

MCSDevice mcs("DXw243yd", "CfxiZLEs0UXcSie4");
MCSDisplayInteger temp("temp"); //溫度
MCSDisplayInteger humid("humid"); //濕度
MCSDisplayInteger soil("soil"); //土壤濕度
MCSControllerOnOff MySwitch("MySwitch"); //風扇手動開關
MCSControllerOnOff MySwitch1("MySwitch1"); //噴水手動開關
MCSControllerAnalog slider("slider"); //風扇自動開關溫度設定slider

void setup()
{
  Serial.begin(9600);
  dht.begin();
  lcd_i2c.begin(16, 2);
  lcd_i2c.clear();
  pinMode(LED_BUILTIN, OUTPUT); //風扇(內建LED)
  pinMode(2, OUTPUT); //噴水(外接LED)

  //新增通道id
  mcs.addChannel(temp);
  mcs.addChannel(humid);
  mcs.addChannel(MySwitch); 
  mcs.addChannel(MySwitch1);
  mcs.addChannel(slider);
  
  Serial.println("Wi-Fi 開始連線");
  while (WiFi.begin(_lwifi_ssid, _lwifi_pass) != WL_CONNECTED) { delay(1000); }
  Serial.println("Wi-Fi 連線成功");
  while(!mcs.connected()) { mcs.connect(); }
  Serial.println("MCS 連線成功");
 }

void loop()
{
  while (!mcs.connected()) {
    mcs.connect();
    if (mcs.connected()) { Serial.println("MCS 已重新連線"); }
  }
  mcs.process(100);

  // 讀取Sensor讀數
  t = dht.readTemperature();
  h = dht.readHumidity();
  s = analogRead(14);
  s = 100 - map(s, 281,4095, 0, 100); //轉換土壤濕度為 0-100
  sprintf(ss, "%3d", s); //轉換土壤濕度為文字格式
  
  // Serial port視窗顯示
  Serial.print("溫度=");
  Serial.println(t);
  Serial.print("濕度=");
  Serial.println(h);
  Serial.print("土壤濕度=");
  Serial.println(s);

  // MCS儀錶板顯示
  temp.set(t);
  humid.set(h);
  soil.set(s);
  
  // 更新開啟風扇溫度
  if (slider.updated()) {
    th = slider.value();
    Serial.print("風扇開啟溫度=");
    Serial.println(th);
  }
  //檢查開關
  if (MySwitch.updated()) {
    if (MySwitch.value()) {f = 1;
        } else {f = 0;}
  }      
  if (MySwitch1.updated()) {
    if (MySwitch1.value()) {w = 1;
        } else {w = 0;}
 }      

  // 風扇控制
  if (t >= th || f == 1 ) digitalWrite(LED_BUILTIN, HIGH);
  if (t < th && f == 0 ) digitalWrite(LED_BUILTIN, LOW);
  // 噴水控制
  if (s <= 50 || w == 1 ) {digitalWrite(2, HIGH);}
  if (s > 50 &&  w== 0 ) {digitalWrite(2, LOW);}
      
  // LCD顯示
  lcd_i2c.setCursor(0,0);
  lcd_i2c.print("T=");
  lcd_i2c.setCursor(2,0);
  lcd_i2c.print(t);
  lcd_i2c.setCursor(4,0);
  lcd_i2c.print((char)223);
  lcd_i2c.setCursor(5,0);
  lcd_i2c.print("C Humid=");
  lcd_i2c.setCursor(13,0);
  lcd_i2c.print(h);
  lcd_i2c.setCursor(15,0);
  lcd_i2c.print("%");
  lcd_i2c.setCursor(0,1);
  lcd_i2c.print("Soil Moist=");
  lcd_i2c.setCursor(11,1);
  lcd_i2c.print(ss);
  lcd_i2c.setCursor(14,1);
  lcd_i2c.print("%");
  
  delay(1000);
}
