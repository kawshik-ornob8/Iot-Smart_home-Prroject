#define buzzar 7
#define flame_sensor_pin A5
#define lpg_pin A1
#define pump 2

void setup() {
 
  pinMode ( flame_sensor_pin , INPUT );
 pinMode ( lpg_pin , INPUT );
  pinMode(buzzar, OUTPUT);
   pinMode(pump, OUTPUT);
}

void loop(){
  digitalWrite ( pump , LOW ) ;
  digitalWrite ( buzzar , LOW ) ;
lpg();
Fire();

}

void Fire(){
  int flame_pin = analogRead ( flame_sensor_pin ) ; // reading from the sensor
if (flame_pin <100) // applying condition
{

digitalWrite ( buzzar , HIGH ) ;// if state is high, then turn high the led
delay(1000);
digitalWrite ( pump , HIGH ) ;
delay(2000);
}
 
}

void lpg(){
  int GAS_VAL = analogRead(lpg_pin);



 if (GAS_VAL > 500)
 {
   digitalWrite ( buzzar , HIGH ) ;
   delay(2000);
  }
  

}
