# Proiect-Practica

Ziua 1 - am fost in permisie
Ziua 2 - mi am ales ctf-uri, am si lucrat de pe PicoCTF 3 ctf-uri: Obedient Cat, Mod 26 si Wave a flag, am mai incercat si altele si cand am vazut ca nu-mi mergm m-am lasat de ele.
Ziua 3 - mi am schimbat tema-> Arduino. Am primit piese si am rulat coduri simple din examples
Ziua 4 - m-am obisnuit cu piesele si placuta si am invatat cum sa folosesc Display-ul I2C, breadboard-ul, senzorul de temperatura, am mers sa-mi cumpar piese de la OptimusDigital dar n-au magazin fizic... asa ca mi le-a trimis tata ziua urmatoare
Ziua 5 - am foarte multe piese si nu stiu ce sa fac cu ele. Am experimentat cu display-ul 4 digits si modulul RTC ca sa fac un ceas. De asemenea, am folosit si o sursa pentru breadboard ca sa pot alimenta montajul de la priza, independent de laptop. Am folosit codul:


-------------------------------------------------------------------------------------------------------------------------------------------
#include "TM1637.h"
 
//{0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20};
//0~9,A,b,C,d,E,F,"-"," ",degree,r,h

#define CLK 9//Pins for TM1637       
#define DIO 8
TM1637 tm1637(CLK,DIO);

// Date and time functions using a DS3231 RTC connected via I2C and Wire lib
#include <Wire.h>
#include "RTClib.h"
RTC_DS3231 rtc;
int hh, mm; 

void setup()
{
  tm1637.init();
  tm1637.set(5); 
  //BRIGHT_TYPICAL = 2,BRIGHT_DARKEST = 0,BRIGHTEST = 7;

  rtc.begin();
// manual adjust
  // January 21, 2014 at 3am you would call:
   //rtc.adjust(DateTime(2022, 6, 20, 10, 8, 0));
// automatic adjust
  //rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
}//end "setup()"
 
void loop(){
DateTime now = rtc.now();
hh = now.hour(), DEC;
mm = now.minute(), DEC;

tm1637.point(POINT_ON);
if ((hh/10) == 0) tm1637.display(0,17);
else
    tm1637.display(0,hh/10);     // hour
    tm1637.display(1,hh%10);
    tm1637.display(2,mm/10);    // minutes
    tm1637.display(3,mm%10);    // 
delay(500);
    tm1637.point(POINT_OFF);
if ((hh/10) == 0) tm1637.display(0,17);
else
    tm1637.display(0,hh/10);     // hour
    tm1637.display(1,hh%10);
    tm1637.display(2,mm/10);    // minutes
    tm1637.display(3,mm%10);    // 
delay(500);
}// end loop() 


------------------------------------------------------------------------------------------------------------------------------------------------------------


