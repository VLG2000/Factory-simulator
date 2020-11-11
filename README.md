# Factory-simulator
Simulates Vision of IOT in factory using temperature and humidity sensor where output is given on the display
/
#include "Arduino.h"
#include<jhd1313m1.h>
#include <th02.hpp>
#include "upm_utilities.h"
#define real
jhd1313m1_context lcd;
int ndx=0;
char str[32];
char strs[32];
char str3[32];
int prev=0;
int prev2=0;
uint8_t rgb[8][3]={
  {0xd1,0x00,0x00},
  {0xff,0x66,0x22},
  {0xff,0xda,0x21},
  {0x33,0xdd,0x00},
  {0x11,0x33,0xcc},
  {0x22,0x00,0x66},
  {0x33,0x00,0x44},
  {0x23,0x34,0xbb}
};

void setup() {
   mraa_add_subplatform(MRAA_GROVEPI,"0");
  #ifdef real
  DebugSerial.begin(9600);
  pinMode(517,OUTPUT);
  #endif
   lcd = jhd1313m1_init(0,0x3e,0x62);
   if(!lcd)
   #ifdef real
   DebugSerial.println("jhd1313m1_i2c_init()failed.");
   #else
    printf("jhd1313m1_i2c_init()failed.");
  #endif
}

void loop() {
  upm::TH02 sensor;
  float temp1=sensor.getTemperature();
  float hum=sensor.getHumidity();
  int sensorValue=analogRead(512);
  int sensorValue1=analogRead(513);
  if ((sensorValue1!=prev)||((prev2-sensorValue)>100))
  {ndx++;
    prev=sensorValue1;
    prev2=sensorValue;
  }
  if(sensorValue1==1023)
  {
      
      snprintf(str,sizeof(str),"Temp %f",temp1);
      jhd1313m1_set_cursor(lcd,0,0);
      jhd1313m1_write(lcd,str,strlen(str));
      uint8_t r=rgb[ndx%8][0];
      uint8_t g=rgb[ndx%8][1];
      uint8_t b=rgb[ndx%8][2];
      jhd1313m1_set_color(lcd,r,g,b);
      snprintf(strs,sizeof(strs),"Humidity %f  ",hum);
      jhd1313m1_set_cursor(lcd,1,0);
      jhd1313m1_write(lcd,strs,strlen(strs));
    if (sensorValue<100)
      {digitalWrite(517,HIGH);}
    else
      {digitalWrite(517,LOW);}
      
  
  }
  else if(sensorValue1==0)
  {
    snprintf(str3,sizeof(str3),"Engine is off   ");
    jhd1313m1_set_cursor(lcd,1,0);
    jhd1313m1_write(lcd,str3,strlen(str3));
    snprintf(str3,sizeof(str3),"                       ");
    jhd1313m1_set_cursor(lcd,0,0);
    jhd1313m1_write(lcd,str3,strlen(str3));
    uint8_t r=rgb[ndx%8][0];
    uint8_t g=rgb[ndx%8][1];
    uint8_t b=rgb[ndx%8][2];
    jhd1313m1_set_color(lcd,r,g,b);
    digitalWrite(517,LOW);
  }
      #ifdef real
      DebugSerial.println(str);
      DebugSerial.println(strs);
      DebugSerial.println(sensorValue);
      DebugSerial.println(prev);
    #endif
      upm_delay(2);
  
}


