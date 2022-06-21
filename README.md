# Proiect-Practica

♫♫♫ Ziua 1 - am fost in permisie


♫♫♫ Ziua 2 - mi am ales ctf-uri, am si lucrat de pe PicoCTF 3 ctf-uri: Obedient Cat, Mod 26 si Wave a flag, am mai incercat si altele si cand am vazut ca nu-mi merg, m-am lasat de ele.


♫♫♫ Ziua 3 - mi am schimbat tema-> Arduino. Am primit piese si am rulat coduri simple din examples. Am folosit display-ul I2C si breadboard-ul si am rulat codul:

---------------------------------------------------------------------------------------------------------------------------------------------------------------
#include <Wire.h> 
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x3f,16,2);  // set the LCD address to 0x27 for a 16 chars and 2 line display

void setup()
{
  lcd.begin();                      // initialize the lcd 
  lcd.begin();
  // Print a message to the LCD.
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print("Hello, world!");
  lcd.setCursor(0,1);
  lcd.print("By Grigore Maria");
  // lcd.setCursor(0,2);
 // lcd.print("Arduino LCM IIC 2004");
  // lcd.setCursor(2,3);
 // lcd.print("Power By Ec-yuan!");
}


void loop()
{
}

---------------------------------------------------------------------------------------------------------------------------------------------------------------



♫♫♫ Ziua 4 - m-am obisnuit cu piesele si placuta si m-am hotarat sa fac un montaj care sa afiseze temperatura => senzor de temperatura. Dar am fortat prea tare un jumper wire pe un pin al senzorului si l am rupt............... (dar am cumparat altul, nu-i problema sper (: ). Cand am vazut asa, am mai cautat alte codulete pe net si am facut un joc cu buton, cu codul:


--------------------------------------------------------------------------------------------------------------------------------------------------------------

#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#define PIN_BUTTON 2
#define PIN_AUTOPLAY 1
#define SPRITE_RUN1 1  //Car sprite
#define SPRITE_RUN2 2
#define SPRITE_JUMP 3
#define SPRITE_JUMP_UPPER '.'         
#define SPRITE_JUMP_LOWER 4
#define SPRITE_TERRAIN_EMPTY ' '      
#define SPRITE_TERRAIN_SOLID 5
#define SPRITE_TERRAIN_SOLID_RIGHT 6
#define SPRITE_TERRAIN_SOLID_LEFT 7
#define CAR_HORIZONTAL_POSITION 1    // Horizontal position of CAR on screen
#define TERRAIN_WIDTH 16
#define TERRAIN_EMPTY 0
#define TERRAIN_LOWER_BLOCK 1
#define TERRAIN_UPPER_BLOCK 2
#define CAR_POSITION_OFF 0          // CAR is invisible
#define CAR_POSITION_RUN_LOWER_1 1  // CAR is running on lower row (pose 1)
#define CAR_POSITION_RUN_LOWER_2 2  //                              (pose 2)
#define CAR_POSITION_JUMP_1 3       // Starting a jump
#define CAR_POSITION_JUMP_2 4       // Half-way up
#define CAR_POSITION_JUMP_3 5       // Jump is on upper row
#define CAR_POSITION_JUMP_4 6       // Jump is on upper row
#define CAR_POSITION_JUMP_5 7       // Jump is on upper row
#define CAR_POSITION_JUMP_6 8       // Jump is on upper row
#define CAR_POSITION_JUMP_7 9       // Half-way down
#define CAR_POSITION_JUMP_8 10      // About to land
#define CAR_POSITION_RUN_UPPER_1 11 // CAR is running on upper row (pose 1)
#define CAR_POSITION_RUN_UPPER_2 12 //                              (pose 2)
LiquidCrystal_I2C lcd(0x3f, 16, 2);
static char terrainUpper[TERRAIN_WIDTH + 1];
static char terrainLower[TERRAIN_WIDTH + 1];
static bool buttonPushed = false;
void initializeGraphics() {
  static byte graphics[] = {
    // Run position 1
    0b00000,
    0b00000,
    0b00100,
    0b11110,
    0b11110,
    0b11111,
    0b01001,
    0b00000,
    // Run position 2
    0b00000,
    0b00000,
    0b00100,
    0b11110,
    0b11110,
    0b11111,
    0b01001,
    0b00000,
    // Jump
    0b00000,
    0b00000,
    0b00100,
    0b11110,
    0b11110,
    0b11111,
    0b01001,
    0b00000,
    // Jump lower
    0b00000,
    0b00000,
    0b00100,
    0b11110,
    0b11110,
    0b11111,
    0b01001,
    0b00000,
    // Ground
    0b01110,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
    // Ground right
    0b01110,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
    // Ground left
    0b01110,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
    0b10101,
    0b11111,
  };
  int i;
  // Skip using character 0, this allows lcd.print() to be used to
  // quickly draw multiple characters
  for (i = 0; i < 7; ++i) {
    lcd.createChar(i + 1, &graphics[i * 8]);
  }
  for (i = 0; i < TERRAIN_WIDTH; ++i) {
    terrainUpper[i] = SPRITE_TERRAIN_EMPTY;
    terrainLower[i] = SPRITE_TERRAIN_EMPTY;
  }
}

// Slide the terrain to the left in half-character increments
//
void advanceTerrain(char* terrain, byte newTerrain) {
  for (int i = 0; i < TERRAIN_WIDTH; ++i) {
    char current = terrain[i];
    char next = (i == TERRAIN_WIDTH - 1) ? newTerrain : terrain[i + 1];
    switch (current) {
      case SPRITE_TERRAIN_EMPTY:
        terrain[i] = (next == SPRITE_TERRAIN_SOLID) ? SPRITE_TERRAIN_SOLID_RIGHT : SPRITE_TERRAIN_EMPTY;
        break;
      case SPRITE_TERRAIN_SOLID:
        terrain[i] = (next == SPRITE_TERRAIN_EMPTY) ? SPRITE_TERRAIN_SOLID_LEFT : SPRITE_TERRAIN_SOLID;
        break;
      case SPRITE_TERRAIN_SOLID_RIGHT:
        terrain[i] = SPRITE_TERRAIN_SOLID;
        break;
      case SPRITE_TERRAIN_SOLID_LEFT:
        terrain[i] = SPRITE_TERRAIN_EMPTY;
        break;
    }
  }
}

bool drawCAR(byte position, char* terrainUpper, char* terrainLower, unsigned int score) {
  bool collide = false;
  char upperSave = terrainUpper[CAR_HORIZONTAL_POSITION];
  char lowerSave = terrainLower[CAR_HORIZONTAL_POSITION];
  byte upper, lower;
  switch (position) {
    case CAR_POSITION_OFF:
      upper = lower = SPRITE_TERRAIN_EMPTY;
      break;
    case CAR_POSITION_RUN_LOWER_1:
      upper = SPRITE_TERRAIN_EMPTY;
      lower = SPRITE_RUN1;
      break;
    case CAR_POSITION_RUN_LOWER_2:
      upper = SPRITE_TERRAIN_EMPTY;
      lower = SPRITE_RUN2;
      break;
    case CAR_POSITION_JUMP_1:
    case CAR_POSITION_JUMP_8:
      upper = SPRITE_TERRAIN_EMPTY;
      lower = SPRITE_JUMP;
      break;
    case CAR_POSITION_JUMP_2:
    case CAR_POSITION_JUMP_7:
      upper = SPRITE_JUMP_UPPER;
      lower = SPRITE_JUMP_LOWER;
      break;
    case CAR_POSITION_JUMP_3:
    case CAR_POSITION_JUMP_4:
    case CAR_POSITION_JUMP_5:
    case CAR_POSITION_JUMP_6:
      upper = SPRITE_JUMP;
      lower = SPRITE_TERRAIN_EMPTY;
      break;
    case CAR_POSITION_RUN_UPPER_1:
      upper = SPRITE_RUN1;
      lower = SPRITE_TERRAIN_EMPTY;
      break;
    case CAR_POSITION_RUN_UPPER_2:
      upper = SPRITE_RUN2;
      lower = SPRITE_TERRAIN_EMPTY;
      break;
  }
  if (upper != ' ') {
    terrainUpper[CAR_HORIZONTAL_POSITION] = upper;
    collide = (upperSave == SPRITE_TERRAIN_EMPTY) ? false : true;
  }
  if (lower != ' ') {
    terrainLower[CAR_HORIZONTAL_POSITION] = lower;
    collide |= (lowerSave == SPRITE_TERRAIN_EMPTY) ? false : true;
  }

  byte digits = (score > 9999) ? 5 : (score > 999) ? 4 : (score > 99) ? 3 : (score > 9) ? 2 : 1;

  // Draw the scene
  terrainUpper[TERRAIN_WIDTH] = '\0';
  terrainLower[TERRAIN_WIDTH] = '\0';
  char temp = terrainUpper[16 - digits];
  terrainUpper[16 - digits] = '\0';
  lcd.setCursor(0, 0);
  lcd.print(terrainUpper);
  terrainUpper[16 - digits] = temp;
  lcd.setCursor(0, 1);
  lcd.print(terrainLower);

  lcd.setCursor(16 - digits, 0);
  lcd.print(score);

  terrainUpper[CAR_HORIZONTAL_POSITION] = upperSave;
  terrainLower[CAR_HORIZONTAL_POSITION] = lowerSave;
  return collide;
}

// Handle the button push as an interrupt
void buttonPush() {
  buttonPushed = true;
}

void setup() {
  pinMode(PIN_BUTTON, INPUT);
  digitalWrite(PIN_BUTTON, HIGH);
  pinMode(PIN_AUTOPLAY, OUTPUT);
  digitalWrite(PIN_AUTOPLAY, HIGH);

  // Digital pin 2 maps to interrupt 0
  attachInterrupt(0/*PIN_BUTTON*/, buttonPush, FALLING);

  initializeGraphics();

  lcd.begin();
  lcd.backlight();
}

void loop() {
  static byte CARPos = CAR_POSITION_RUN_LOWER_1;
  static byte newTerrainType = TERRAIN_EMPTY;
  static byte newTerrainDuration = 1;
  static bool playing = false;
  static bool blink = false;
  static unsigned int distance = 0;

  if (!playing) {
    drawCAR((blink) ? CAR_POSITION_OFF : CARPos, terrainUpper, terrainLower, distance >> 3);
    if (blink) {
      lcd.setCursor(0, 0);
      lcd.print("Press Start");
    }
    delay(100);
    blink = !blink;
    if (buttonPushed) {
      initializeGraphics();
      CARPos = CAR_POSITION_RUN_LOWER_1;
      playing = true;
      buttonPushed = false;
      distance = 0;
    }
    return;
  }

  // Shift the terrain to the left
  advanceTerrain(terrainLower, newTerrainType == TERRAIN_LOWER_BLOCK ? SPRITE_TERRAIN_SOLID : SPRITE_TERRAIN_EMPTY);
  advanceTerrain(terrainUpper, newTerrainType == TERRAIN_UPPER_BLOCK ? SPRITE_TERRAIN_SOLID : SPRITE_TERRAIN_EMPTY);

  // Make new terrain to enter on the right
  if (--newTerrainDuration == 0) {
    if (newTerrainType == TERRAIN_EMPTY) {
      newTerrainType = (random(3) == 0) ? TERRAIN_UPPER_BLOCK : TERRAIN_LOWER_BLOCK;
      newTerrainDuration = 10 + random(6);
    } else {
      newTerrainType = TERRAIN_EMPTY;
      newTerrainDuration = 10 + random(6);
    }
  }

  if (buttonPushed) {
    if (CARPos <= CAR_POSITION_RUN_LOWER_2) CARPos = CAR_POSITION_JUMP_1;
    buttonPushed = false;
  }

  if (drawCAR(CARPos, terrainUpper, terrainLower, distance >> 3)) {
    playing = false; // The CAR collided with something. Too bad.
    for (int i = 0; i <= 2; i++) {
    }
  } else {
    if (CARPos == CAR_POSITION_RUN_LOWER_2 || CARPos == CAR_POSITION_JUMP_8) {
      CARPos = CAR_POSITION_RUN_LOWER_1;
    } else if ((CARPos >= CAR_POSITION_JUMP_3 && CARPos <= CAR_POSITION_JUMP_5) && terrainLower[CAR_HORIZONTAL_POSITION] != SPRITE_TERRAIN_EMPTY) {
      CARPos = CAR_POSITION_RUN_UPPER_1;
    } else if (CARPos >= CAR_POSITION_RUN_UPPER_1 && terrainLower[CAR_HORIZONTAL_POSITION] == SPRITE_TERRAIN_EMPTY) {
      CARPos = CAR_POSITION_JUMP_5;
    } else if (CARPos == CAR_POSITION_RUN_UPPER_2) {
      CARPos = CAR_POSITION_RUN_UPPER_1;
    } else {
      ++CARPos;
    }
    ++distance;

    digitalWrite(PIN_AUTOPLAY, terrainLower[CAR_HORIZONTAL_POSITION + 2] == SPRITE_TERRAIN_EMPTY ? HIGH : LOW);
  }
}


--------------------------------------------------------------------------------------------------------------------------------------------------------------

Am mers sa-mi cumpar si alte piese de la OptimusDigital dar n-au magazin fizic... asa ca mi le-a trimis tata ziua urmatoare



♫♫♫ Ziua 5 - am foarte multe piese si nu stiu ce sa fac cu ele. Am experimentat cu display-ul 4 digits si modulul RTC ca sa fac un ceas. De asemenea, am folosit si o sursa pentru breadboard ca sa pot alimenta montajul de la priza, independent de laptop. Am folosit codul:


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


♫♫♫ Ziua 6 - Am facut un montaj care afiseaza temperatura, umiditatea si indicele de caldura (temperatura simtita de corpul uman) folosind un display OLED si un senzor DHT22 de temperatura si umiditate. Cod folosit:

------------------------------------------------------------------------------------------------------------------------------------------------------------

#include "U8glib.h"
#include "DHT.h"
#define DHTPIN 2 
#define DHTTYPE DHT22 
DHT dht(DHTPIN, DHTTYPE); 
U8GLIB_SH1106_128X64 u8g(13, 11, 10, 9, 8);  // D0=13, D1=11, CS=10, DC=9, Reset=8
void setup() {
  dht.begin(); 
  u8g.firstPage();  
  do {
    u8g.setFont(u8g_font_helvB10);  
    u8g.drawStr(30, 10, "Welcome "); 
    u8g.drawStr(50, 30, "To ");
    u8g.drawStr(10, 50, "ElectronicsHob");
    u8g.drawStr(10, 60, "byists.com");
  } while( u8g.nextPage() );
  delay(5000);  
}
void loop() {
  float hum = dht.readHumidity(); 
  float temp = dht.readTemperature(); 
  float fah = dht.readTemperature(true);
  if (isnan(hum) || isnan(temp) || isnan(fah)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  float heat_index = dht.computeHeatIndex(fah, hum);
  float heat_indexC = dht.convertFtoC(heat_index);    //Calculating the heat index in Celsius
  
  u8g.firstPage();  
  do {
    u8g.setFont(u8g_font_helvR12);  
    u8g.drawStr(0, 15, "Temp:");  
    u8g.setPrintPos(75, 15);
    u8g.print(temp, 0);
    u8g.print((char)176);
    u8g.print("C");
    
    u8g.drawStr(0, 35, "Humi:");
    u8g.setPrintPos(75, 35);
    u8g.print(hum, 0);
    u8g.print("%");
    
    u8g.drawStr(0, 55, "Hi:");
    u8g.setPrintPos(75, 55);
    u8g.print(heat_indexC, 0);
    u8g.print("%");
  } while( u8g.nextPage() );
  delay(5000);
}

-----------------------------------------------------------------------------------------------------------------------------------------------------

-Am facut un montaj cu un fotorezistor-> cand detecteaza lumina, ledul se aprinde. Cand nu, ledul se stinge. Cod folosit:

--------------------------------------------------------------------------------------------------------------------------------------------------------
const int sensorPin = 0;
const int ledPin = 9;

int lightLevel, high = 0, low = 1023;


void setup()
{
  pinMode(ledPin, OUTPUT);
}


void loop()
{
  lightLevel = analogRead(sensorPin);
  manualTune();  // manually change the range from light to dark
  analogWrite(ledPin, lightLevel);

}


void manualTune()
{

  lightLevel = map(lightLevel, 350, 800, 0, 255);
  lightLevel = constrain(lightLevel, 0, 255);
} 


void autoTune()
{
  
  if (lightLevel < low)
  {
    low = lightLevel;
  }

  if (lightLevel > high)
  {
    high = lightLevel;
  }


  lightLevel = map(lightLevel, low+10, high-10, 0, 255);
  lightLevel = constrain(lightLevel, 0, 255);


}

--------------------------------------------------------------------------------------------------------------------------------------------------------

♫♫♫ Ziua 7 - Am mai facut un montaj cu senzorul BMP180 de presiune atmosferica: Se afiseaza pe un display I2C temperatura si presiunea atmosferica sub forma de loading bar. Video youtube: https://www.youtube.com/watch?v=0IojNYgDD7U . Cod folosit:

---------------------------------------------------------------------------------------------------------------------------------------------------------

#include <SFE_BMP180.h>
#include <Wire.h>


SFE_BMP180 rjt;// creating object called "rjt" (robojax temperature)
#define UNIT 'C' // unit type C for Celsius F for Fahrenheit
#define maxTemperature 40 // is the value in Celsius or Fahrenheit set in line above

// LCD settings
#include <LiquidCrystal_I2C.h>
byte lcdNumCols = 16; // -- number of columns in the LCD
byte lcdLine = 2; // -- number of line in the LCD

LiquidCrystal_I2C lcd(0x3f,lcdNumCols,lcdLine); //0x3f is address for I2C
                  // to get I2C address, run I2C Scanner. 
                  //Link is provided (in same page as this code) at http://robojax.com/learn/arduino
// bargraph settings
#include <LcdBarGraphRobojax.h>
LcdBarGraphRobojax robojax(&lcd, 16, 0, 0);  // -- creating 16 character long bargraph starting at char 0 of line 0 (see video)



void setup()
{
  Serial.begin(9600);//initialize serial monitor
 // -- initializing the LCD
  lcd.begin();
  lcd.clear();
  lcd.print("Robojax"); 
   
 
  // Initialize the sensor (it is important to get calibration values stored on the device).

  if (rjt.begin())
  {
   lcd.setCursor (0,1); //  
    lcd.print("BMP180 Bargraph");  
  }else {
    lcd.setCursor (0,1); //  
    lcd.print("BMP180 Failed"); 
    while(1); // Pause forever.
  }
  // -- give use some time to read the text above
  delay(2000);
  lcd.clear();  
}// setup

void loop()
{
 // Robojax BMP180 Bargraph main loop

 robojax.clearLine(1);// clear line 1 to display fresh temperature
 float T = getTemp();// get the temperature
 float Tgraph=T;
 if(T !=999){
     if( Tgraph > maxTemperature){   
        Tgraph =0;
     }
        robojax.drawValue( Tgraph, maxTemperature);// draw the bargraph     
      // Print out the measurement:
      Serial.print("temperature: ");
      Serial.print(T,2);


      lcd.setCursor (0,1); //
      if(T< maxTemperature){       
       lcd.print("Temp.:"); 
      }else{
       lcd.print("Max.:"); 
      }
      lcd.setCursor (7,1); //
      lcd.print(T); // print  
      lcd.print((char)223);// prints degree symbol
      if(UNIT =='C'){
        lcd.print("C");//
        Serial.println(" deg C");         
      }else{
        lcd.print("F");
        Serial.println(" deg F");         
      }
      
  }else{
    Serial.print("Status error");
      lcd.setCursor (0,1); //
      lcd.print("Error ");     
  }

delay(500);
}

 ////////////////////////////////////////////
double getTemp(){
    char status;
  double Temp,newTemp;
 
status = rjt.startTemperature();
  if (status != 0)
  {
    // Wait for the measurement to complete:
    delay(status);

    // Retrieve the completed temperature measurement:
    // Note that the measurement is stored in the variable T.
    // Function returns 1 if successful, 0 if failure.

    status = rjt.getTemperature(Temp);
    if (status != 0)
    {  
      if (UNIT =='F')
      {
        newTemp = (9.0/5.0)*Temp+32.0;// convert to degrees farenheit
      }else{
        newTemp = Temp;// keeps degree celsius
      }
    }
    return newTemp;    
  }else{
    return 999;// if error, then retuen 999
  }
}//getTemp/////////////////////////////


------------------------------------------------------------------------------------------------------------------------------------------------------------------

-Am folosit un modul encoder rotativ (Rotary encoder) pentru a face un montaj care afiseaza in Serial Monitor directia de rotatie a butonului cat si numarul de unitati cu care s-a rotit. Cod folosit:

----------------------------------------------------------------------------------------------------------------------------------------------------------------

// Rotary Encoder Inputs
#define CLK 2
#define DT 3
#define SW 4

int counter = 0;
int currentStateCLK;
int lastStateCLK;
String currentDir ="";
unsigned long lastButtonPress = 0;

void setup() {
  
  // Set encoder pins as inputs
  pinMode(CLK,INPUT);
  pinMode(DT,INPUT);
  pinMode(SW, INPUT_PULLUP);

  // Setup Serial Monitor
  Serial.begin(9600);

  // Read the initial state of CLK
  lastStateCLK = digitalRead(CLK);
}

void loop() {
  
  // Read the current state of CLK
  currentStateCLK = digitalRead(CLK);

  // If last and current state of CLK are different, then pulse occurred
  // React to only 1 state change to avoid double count
  if (currentStateCLK != lastStateCLK  && currentStateCLK == 1){

    // If the DT state is different than the CLK state then
    // the encoder is rotating CCW so decrement
    if (digitalRead(DT) != currentStateCLK) {
      counter --;
      currentDir ="CCW";
    } else {
      // Encoder is rotating CW so increment
      counter ++;
      currentDir ="CW";
    }

    Serial.print("Direction: ");
    Serial.print(currentDir);
    Serial.print(" | Counter: ");
    Serial.println(counter);
  }

  // Remember last CLK state
  lastStateCLK = currentStateCLK;

  // Read the button state
  int btnState = digitalRead(SW);

  //If we detect LOW signal, button is pressed
  if (btnState == LOW) {
    //if 50ms have passed since last LOW pulse, it means that the
    //button has been pressed, released and pressed again
    if (millis() - lastButtonPress > 50) {
      Serial.println("Button pressed!");
    }

    // Remember last button press event
    lastButtonPress = millis();
  }

  // Put in a slight delay to help debounce the reading
  delay(1);
}

---------------------------------------------------------------------------------------------------------------------------------------------------------------------


♫♫♫ Ziua 8 - Incurajata si de tatal meu si avand toate piesele necesare, m-am hotarat sa fac un radio functional.
Piesele necesare:
-Placa plexiglas pentru montarea a doua placi beadboard full-sized din cauza multitudinii de module si fire
-2 breadboard-uri full-sized
-Modul Radio-FM, numit RDA 5807M
-Translator de nivel logic (traduce nivelul semnalelor logice trimise de placuta Arduino de la 5V la 3.3V => nu ard placuta Radio)
-Modul amplificator, numit PAM8403
-Display cu interfata I2C
-2 difuzoare
-sursa de alimentare a breadboard-ului; doresc ca radio-ul sa functioneze independent de laptop
-alimentator
-antena ~70 cm
-jumper wires
-4 butoane: Volume Up, Volume Down, Frequency Up, Frequency Down




