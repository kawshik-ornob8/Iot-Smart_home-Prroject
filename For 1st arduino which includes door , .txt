

// RemoteXY select connection mode and include library
#define REMOTEXY_MODE__ESP8266_HARDSERIAL_POINT

#include <RemoteXY.h>
#include <Keypad.h>

#include <LiquidCrystal_I2C.h>

#include <Servo.h>


// RemoteXY connection settings
#define REMOTEXY_SERIAL Serial
#define REMOTEXY_SERIAL_SPEED 115200
#define REMOTEXY_WIFI_SSID "RemoteXY"
#define REMOTEXY_WIFI_PASSWORD "12345678"
#define REMOTEXY_SERVER_PORT 6377


// RemoteXY configurate
#pragma pack(push, 1)
uint8_t RemoteXY_CONF[] =  // 261 bytes
  { 255, 2, 0, 2, 0, 254, 0, 16, 134, 0, 2, 1, 61, 33, 22, 11, 2, 26, 31, 1,
    70, 65, 78, 32, 79, 78, 0, 70, 65, 78, 32, 79, 70, 70, 0, 2, 1, 11, 33, 22,
    11, 2, 26, 31, 31, 76, 105, 103, 104, 116, 32, 79, 78, 0, 76, 105, 103, 104, 116, 32,
    79, 70, 70, 0, 69, 1, 2, 3, 10, 10, 129, 0, 31, 6, 39, 8, 1, 68, 101, 97,
    100, 72, 101, 97, 100, 0, 129, 0, 89, 3, 18, 3, 31, 80, 114, 111, 106, 101, 99, 116,
    32, 84, 101, 97, 109, 58, 0, 129, 0, 89, 6, 21, 2, 31, 80, 114, 111, 110, 111, 121,
    32, 75, 117, 109, 97, 114, 32, 77, 111, 110, 100, 97, 108, 0, 129, 0, 89, 8, 22, 2,
    31, 75, 97, 119, 115, 104, 105, 107, 32, 65, 104, 109, 101, 100, 32, 79, 114, 110, 111, 98,
    0, 129, 0, 89, 10, 22, 2, 31, 77, 100, 46, 32, 77, 97, 115, 117, 100, 32, 82, 97,
    110, 97, 0, 129, 0, 89, 12, 22, 2, 31, 66, 97, 98, 105, 107, 97, 110, 97, 100, 111,
    32, 82, 111, 121, 32, 83, 104, 117, 118, 111, 0, 129, 0, 89, 14, 22, 2, 31, 77, 97,
    104, 97, 100, 105, 32, 72, 97, 115, 97, 110, 32, 83, 104, 97, 110, 117, 0, 129, 0, 89,
    16, 18, 2, 31, 84, 97, 115, 110, 117, 118, 97, 32, 77, 117, 104, 116, 97, 115, 105, 109,
    0 };

// this structure defines all the variables and events of your control interface
struct {

  // input variables
  uint8_t switch_1;  // =1 if switch ON and =0 if OFF
  uint8_t switch_2;  // =1 if switch ON and =0 if OFF

  // output variables
  int16_t sound_1;  // =0 no sound, else ID of sound, =1001 for example, look sound list in app

  // other variable
  uint8_t connect_flag;  // =1 if wire connected, else =0

} RemoteXY;
#pragma pack(pop)

/////////////////////////////////////////////
//           END RemoteXY include          //
/////////////////////////////////////////////

#define PIN_SWITCH_1 A1
#define PIN_SWITCH_2 A0

#define SENSOR_PIN_G 4



Servo myservom;

Servo myservog;


char key;

char code[] = { '1', '2', '3', '4' };  //The default code, you can change it or make it a 'n' digits one

char check1[sizeof(code)];  //Where the new key is stored
char check2[sizeof(code)];  //Where the new key is stored again so it's compared to the previous one

int a = 0, i = 0, j = 0;
int flame_pin = HIGH;
// initialize the LCD

//Temperature Sensor initialize

float temp;


LiquidCrystal_I2C lcd(0x27, 16, 2);


const byte rows = 4;  // set display to 4 rows

const byte cols = 4;  // set display to 3 columns

char keys[rows][cols] = {
  { '1', '2', '3', 'A' },
  { '4', '5', '6', 'B' },
  { '7', '8', '9', 'C' },
  { '*', '0', '#', 'D' }
};

byte rowPins[rows] = { 13, 12, 11, 10 };

byte colPins[cols] = { 9, 8, 7, 6 };

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, rows, cols);


void setup() {
  RemoteXY_Init();

  pinMode(PIN_SWITCH_1, OUTPUT);
  pinMode(PIN_SWITCH_2, OUTPUT);

  // TODO you setup code

  lcd.init();
  lcd.clear();
  lcd.backlight();

  lcd.begin(16, 2);
  lcd.setCursor(2, 0);
  lcd.print("-Welcome To-");
  lcd.setCursor(3, 1);
  lcd.print("-DeadHead-");
  delay(5000);
   lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Press B ToUnlock");
  lcd.setCursor(0, 1);
  lcd.print("TEMPERATURE: ");
  myservom.attach(3);
  myservog.attach(5);
  pinMode(SENSOR_PIN_G, INPUT);
}

void loop() {
  RemoteXY_Handler();

  digitalWrite(PIN_SWITCH_1, (RemoteXY.switch_1 == 0) ? LOW : HIGH);
  digitalWrite(PIN_SWITCH_2, (RemoteXY.switch_2 == 0) ? LOW : HIGH);
   
  // TODO you loop code
  // use the RemoteXY structure for data transfer
  // do not call delay()
  temparature();
  main_Door();
  G_motion();
}
void temparature(){
  temp = analogRead(A2);
   // read analog volt from sensor and save to variable temp
  float volts = ((temp * 500) / 1023.0);
 
   // convert the analog volt to its temperature equivalent
 
  lcd.setCursor(12, 1);
  lcd.print(volts);
  delay(500);
}
void main_Door() {
  key = keypad.getKey();
  myservom.write(5);

  if (key == 'B') {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Press A To Enter");
    delay(2000);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Enter Password:");

    ReadPassword();
    if (a == sizeof(code)) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Unlocked!");
      myservom.write(90);
      delay(1000);
      lcd.clear();
      lcd.print("Press B ToUnlock");
      lcd.setCursor(0, 1);
      lcd.print("TEMPERATURE: ");
      temparature();
    } else {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Wrong Password!");
      delay(3000);
      lcd.clear();
      lcd.print("Press B ToUnlock");
      lcd.setCursor(0, 1);
      lcd.print("TEMPERATURE: ");
      temparature();
    }
  }
}

void ReadPassword() {
  a = 0;
  i = 0;
  j = 0;
  while (key != 'A') {
    key = keypad.getKey();
    if (key != NO_KEY && key != 'A') {
      lcd.setCursor(j, 1);
      lcd.print("*");
      j++;
      if (key == code[i] && i < sizeof(code)) {
        a++;
        i++;
      } else {
        a--;
      }
    }
  }
  //key=NO_KEY;
}
void G_motion() {
  myservog.write(0);
  int sensorValue2 = digitalRead(SENSOR_PIN_G);
  //Serial.println(sensorValue);
  if (sensorValue2 == HIGH) {

    myservog.write(90);
    delay(1000);


  } else {

    myservog.write(0);
  }
}
