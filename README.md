#include <LiquidCrystal.h>
LiquidCrystal lcd(13, 12, 11, 4, 3, 8);

int charge_r=6;
int pump=7;

int soil_moisture, rain_sensor, temperature, ph, a=1;
float vout_B = 0.0;
float vin_B = 0.0;
float vout_S = 0.0;
float vin_S = 0.0;
float R1_B = 56000.0; 
float R2_B = 33000.0; 
float R1_S = 56000.0; 
float R2_S = 33000.0;
int value_B, value_S;
int h1,h2,m1,m2,s1,s2,p1,p2;

char rec[50];
int val=0;
boolean started = true;
int receive=1;

void setup() 
{
  Serial.begin(9600);    
  Serial1.begin(9600);    
  lcd.begin (20, 4);
  lcd.setCursor(0,1); lcd.print(" SMART  AGRICULTURE ");
  lcd.setCursor(0,2); lcd.print("     MANAGEMENT     ");
  pinMode(charge_r, OUTPUT);
  digitalWrite(charge_r, HIGH);
  pinMode(pump, OUTPUT);
  digitalWrite(pump, HIGH);
  delay(1000);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("PLEASE WAIT...     ");
  lcd.setCursor(0,1);
  lcd.print("GETTING RTC TIME...");
  delay(2000);
  lcd.clear();
  delay(5);
}

void loop()
{
  lcd.setCursor(0,0); lcd.print(" SMART AGRICULTURE");
  
  now = rtc.now();
  showTime();
  soil_moisture=1023-analogRead(A0);
  rain_sensor=1023-analogRead(A1);
  temperature=analogRead(A2)-40;

  value_B= analogRead(A3);
  vout_B = (value_B * 5.0) / 1024.0; 
  vin_B = vout_B / (R2_B/(R1_B+R2_B)); 
    
  Serial.print("Battery_Volt: ");
  Serial.print(vin_B); 
  Serial.println("V");
    
  value_S= analogRead(A4);
  vout_S = (value_S * 5.0) / 1024.0; 
  vin_S = vout_S / (R2_S/(R1_S+R2_S)); 
  Serial.print("Supply_Volt: ");
  Serial.print(vin_S); 
  Serial.println("V");

       if(vin_B>8.0 && vin_S<1) { batterylevel(19,1); }
  else if(vin_S>10) {lcd.setCursor(16,1); lcd.print("    ");} 
  
  if(vin_S>10 && (vin_B>6.0 && vin_B<=8.0)) {digitalWrite(charge_r, LOW);} 
  else {digitalWrite(charge_r, HIGH);}
    
  lcd.setCursor(0,2); lcd.print("Mois:");lcd.setCursor(5,2); 
       if(soil_moisture<10)   {lcd.print(soil_moisture);lcd.print("   ");}
  else if(soil_moisture<100)  {lcd.print(soil_moisture);lcd.print("  ");}
  else if(soil_moisture<1000) {lcd.print(soil_moisture);lcd.print(" ");}
  
  lcd.setCursor(10,2); lcd.print("Rain:");lcd.setCursor(15,2); 
       if(rain_sensor<10)   {lcd.print(rain_sensor);lcd.print("   ");}
  else if(rain_sensor<100)  {lcd.print(rain_sensor);lcd.print("  ");}
  else if(rain_sensor<1000) {lcd.print(rain_sensor);lcd.print(" ");}

  lcd.setCursor(10,3); lcd.print("Temp:");lcd.setCursor(15,3);lcd.print(temperature); 
       //if(temperature<10)   {lcd.print(temperature);lcd.print("   ");}
  //else if(temperature<100)  {lcd.print(temperature);lcd.print("  ");}
  //else if(temperature<1000) {lcd.print(temperature);lcd.print(" ");}

  lcd.setCursor(0,3);
  lcd.print("pH:");
  lcd.setCursor(3,3);
  lcd.print(rec[3]);lcd.print(rec[4]);
  Serial.print("ph:");Serial.print(rec[3]);Serial.println(rec[4]);
  val=0;
  delay(100);

  if(soil_moisture<=400)
  {
    digitalWrite(pump,LOW);
  }
  else 
  {
    digitalWrite(pump,HIGH);
  }
  if(soil_moisture<=400)
  {
    if(a==1)
    {
      lcd.clear();
      lcd.setCursor(0,0); lcd.print(" SMART AGRICULTURE");
      lcd.setCursor(0,2);  lcd.print("Please Wait...    ");
      lcd.setCursor(0,3);  lcd.print("Sending SMS...    ");
      Serial.println("Sending SMS...    ");
      Serial1.print("AT+CMGS=\"+919944204570\"\r");//+919944204570
      delay(1500);
      Serial1.println("Smart Agriculture Management");
      Serial1.print("Moisture:");
      Serial1.println(soil_moisture);
      Serial1.print("Rain:");
      Serial1.println(rain_sensor);
      Serial1.print("Temperature:");
      Serial1.println(temperature);
      Serial1.print("pH:");
      Serial1.print(rec[3]);Serial1.println(rec[4]);
      delay(1000);
      Serial1.println((char)26);     
      delay(1000);     
      Serial1.print("AT+CMGS=\"+919345440375\"\r");//+919345440375
      delay(1500);
      Serial1.println("Smart Agriculture Management");
      Serial1.print("Moisture:");
      Serial1.println(soil_moisture);
      Serial1.print("Rain:");
      Serial1.println(rain_sensor);
      Serial1.print("Temperature:");
      Serial1.println(temperature);
      Serial1.print("pH:");
      Serial1.print(rec[3]);Serial1.println(rec[4]);
      delay(1000);
      Serial1.println((char)26);     
      delay(1000);    
      delay(100);
      lcd.clear();
      a=0;
    }
  }
  else 
  {
    a=1;
  }
  Serial.print(soil_moisture);
  Serial.print(":");
  Serial.print(rain_sensor);
  Serial.print(":");
  Serial.print(temperature);
  Serial.print(":");
  Serial.print(rec[3]);
  Serial.println(rec[4]);
  delay(500);
  delay(3000);
}
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
void lora_send()
{
  Serial.println("Lora Sending...");
  //#0000:0000:0000:00*00:00,
  //#1023:1023:0027:37*08:10,//mois:rain:temp:pH*time_Hour:time_minute
  LoRa.print('#');
  Serial_decimal4(soil_moisture);
  LoRa.print(':');
  Serial_decimal4(rain_sensor);
  LoRa.print(':');
  Serial_decimal4(temperature);
  LoRa.print(':');
  LoRa.print(ph);
  LoRa.print(':');
  LoRa.print(h1);LoRa.print(h2);
  LoRa.print(':');
  LoRa.print(m1);LoRa.print(m2);
  LoRa.print(',');

  Serial.print('#');
  Serial_decimal4s(soil_moisture);
  Serial.print(':');
  Serial_decimal4s(rain_sensor);
  Serial.print(':');
  Serial_decimal4s(temperature);
  Serial.print(':');
  Serial.print(ph);
  Serial.print(':');
  Serial.print(h1);LoRa.print(h2);
  Serial.print(':');
  Serial.print(m1);LoRa.print(m2);
  Serial.print(',');
}
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
void showTime()
{
    lcd.setCursor(7,1);
    lcd.print(now.hour());
    lcd.print(':');
    lcd.print(now.minute());
    
    lcd.print("   ");

    h1=now.hour()/10;
    h2=now.hour()%10;
    m1=now.minute()/10;
    m2=now.minute()%10;
    s1=now.second()/10;
    s2=now.second()%10;
    
    Serial.print("RTC Time:");
    Serial.print(now.hour());
    Serial.print(':');
    Serial.print(now.minute());
    Serial.print(':');
    Serial.print(now.second());
    Serial.println(" ");

    Serial.print("RTC Time:");
    Serial.print(h1);Serial.print(h2);
    Serial.print(':');
    Serial.print(m1);Serial.print(m2);
    Serial.print(':');
    Serial.print(s1);Serial.print(s2);
    Serial.println(" ");
}
///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
void serialEvent() 
{
 while(Serial.available()) 
   {
      rec[val]=(char)Serial.read();
      if (rec[0]=='1')
      {rec++;}
   }
   }
