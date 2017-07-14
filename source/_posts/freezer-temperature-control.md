---
title: Freezer Temperature Control.
date: 2017-07-11 20:00:00
tags: kegerator, keezer, beer
---

{% asset_img temperature-control.png 38 Degrees%}


I don't want beer-cycles (although it might be interesting to try) so I need a way to regulate the temperature on my Keezer.  Many people use the <a target="_blank" href="https://www.amazon.com/gp/product/B011296704/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B011296704&linkCode=as2&tag=samkristoff-20&linkId=d887d015a837b61cf2144f9deb1b1019">Itc-308 Digital Temperature Controller</a> as a quick and easy way to convert a freezer into a refrigerator.  I, however, opted for a bit more DIY solution which will allow me to integrate the temperature control into the tablet user interface and have more control over the entire system.

### The Controller
{% asset_img teensy-pre-solder.jpg Teensy 3.2%}
I decided to use a <a target="_blank" href="https://www.amazon.com/gp/product/B015M3K5NG/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B015M3K5NG&linkCode=as2&tag=samkristoff-20&linkId=40ba9b1a98676ba627857f9454fd03a3">Teensy 3.2</a> to handle all of the control signal on my Keezer.  This includes monitoring temperature, turning the freezer on and off, talking to the tablet, controlling beer flow and more.  The Teensy has plenty of I/O and processing power to keep the Keezer running.

### Temperature Sensors
{% asset_img ds18b20-temperature-probe.jpg DS18B20 Temperature Probe%}
Temperature changes relatively slowly over time and while I don't want my beer getting warm, I also don't need to know it's exact temperature to the millidegree.  The <a target="_blank" href="https://www.amazon.com/gp/product/B01DQQPR2A/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B01DQQPR2A&linkCode=as2&tag=samkristoff-20&linkId=c1479bcef02a55085069730c91cdd5fe">ELENKER Waterproof Temperature Probe</a> can measure -10° to 85° C in about .5° C increments which is more than enough for the Keezer.  Best of all they're inexpensive and super easy to use with Teensy!

### Power Control
{% asset_img powerswitch-tail.jpg DS18B20 PowerSwitch Tail II%}
Although they're getting hard to find the <a target="_blank" href="https://www.amazon.com/gp/product/B00B888VHM/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B00B888VHM&linkCode=as2&tag=samkristoff-20&linkId=1df647d959134c8c05da3f4793851483">PowerSwitch Tail II</a> is still my favorite relay for 'wall power' devices.  No cutting cables or external circuitry required, just plug the freezer (or device of your choice) into one end and the other end into the wall, then turn the device on and off by toggling a low voltage digital output.  Simple and safe if not the most cost effective.

### Bringing It All Together
[Apparently](http://bfy.tw/CqWR) the optimal beer keg temperature is 38° F.  I don't want the freezer compressor turning on and off too often so I'm going to shoot for 37-39° F.  The example code below reads from the temperature sensor and turns the freezer on if the temperature is above the high set point (39° F) or turns the freezer off if the temperature is below the low set point (37° F). 

```C++
#include <OneWire.h>

//Configuration
#define TEMP_PROBE_CHAN 10
#define MAIN_TEMP_PROBE_ADDR 0x28FFD723A01646E0

#define COOL_CHAN 13
#define KEEZER_TEMP_MAX 39
#define KEEZER_TEMP_MIN 37

//Global Variables
//Initialize oneWire on DIO 10 to read from DS18B20 temperature sensors
OneWire ow(TEMP_PROBE_CHAN);
byte mainTempProbeAddr[8] = {0x28, 0xFF, 0xD7, 0x23, 0xA0, 0x16, 0x04, 0x6E};

void setup() {
  Serial.begin(9600);
  
  //Set PowerSwitch Tail DIO pin as output.
  pinMode(COOL_CHAN, OUTPUT);

  //Print all OneWire device addresses
  printOneWireDeviceAddresses();
}

void loop() {  
  float temp = readTempProbe(mainTempProbeAddr);
  //Check to make sure temp is valid (If temp read fails it will return -1000)
  if(temp > -1000)
  {
     updateCoolingMode(temp);    
  } 
  delay(1000);
}

//Read temperature from the DS18B20
float readTempProbe(byte* address)
{  
  //Initialize communication
  ow.reset();
  ow.select(address);
  ow.write(0x44, 1);
  delay(1000); 
  ow.reset(); 
  ow.select(address);    
  ow.write(0xBE);  

  //Read Data
  byte data[12];
  for(int i = 0; i < 9; i++) {
    data[i] = ow.read();   
  }

  if(ow.crc8(data, 8) != data[8])
  {
    Serial.println("Failed CRC");
    return -1000;
  }

  //Convert data to temp
  int16_t raw = (data[1] << 8) | data[0];  
  float celsius = (float)raw / 16.0;
  float fahrenheit = celsius * 1.8 + 32.0; 
  Serial.print(fahrenheit);
  Serial.println(" F");
  return fahrenheit;
}

//Update the refridgeration state based on the specified current temperature in F.  This function returns true when cooling is enabled and false otherwise.
bool updateCoolingMode(float tempF)
{
  //Display Freezer Mode
  if (tempF > KEEZER_TEMP_MAX)
  {
    Serial.println("Keezer Cooling");
    digitalWrite(COOL_CHAN, HIGH);
    return true;
  }
  else if (tempF < KEEZER_TEMP_MIN)
  {
    Serial.println("Keezer Idle");
    digitalWrite(COOL_CHAN, LOW);
    return false;
  }
  return false;
}

//Print All OneWire devices addresses on the bus.
void printOneWireDeviceAddresses()
{
  byte addr[8];
  while(ow.search(addr)) 
  {
    Serial.print("Address - 0x");
    for (int i = 0; i < 8; i++) 
    {   
      if(addr[i] > 0xF) {
        Serial.print(addr[i], HEX);  
      }
      else {
        Serial.print("0");  
        Serial.print(addr[i], HEX);  
      }      
    }
  }
  Serial.println();
}
```
Based on [the example](https://www.pjrc.com/teensy/td_libs_OneWire.html) from PJRC.