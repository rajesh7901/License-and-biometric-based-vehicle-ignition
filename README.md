# License-and-biometric-based-vehicle-ignition
//Start the vehicle by verifying the driving license , biometrics and alcohol contents

#include <Wire.h> 
#include <LiquidCrystal_I2C.h>                  //LCD Display library
LiquidCrystal_I2C lcd(0x27, 20,4);
//**********
#include <SPI.h>
#include <MFRC522.h>                        //RFID library
#define RST_PIN         5                  // Configurable, see typical pin layout above
#define SS_PIN          53                // Configurable, see typical pin layout above
MFRC522 mfrc522(SS_PIN, RST_PIN);        // Create MFRC522 instance
//******************
#include <Adafruit_Fingerprint.h>
#if (defined(__AVR__) || defined(ESP8266)) && !defined(__AVR_ATmega2560__)
  SoftwareSerial mySerial(18, 19);
#else
  #define mySerial Serial1
#endif
  Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

uint8_t id;
#define sensor A15               //Gas sensor pin

int System_on=27;             //LED's for indication of all the STEPs 
int System_off=26;
int Rfid_off=30;
int Rfid_on=31;
int Fingerprint_on=25;
int Fingerprint_off=24;
int Gas_on=22;
int Gas_off=23;
int Seatbelt_off=28; 
int Seatbelt_on=29; 

void setup()
{
 Serial.begin(9600);                       // Initialize serial communications with the PC

                        // LED's pinmode's
 pinMode(22, OUTPUT);
 pinMode(23, OUTPUT);
 pinMode(24, OUTPUT);
 pinMode(25, OUTPUT);
 pinMode(26, OUTPUT);
 pinMode(27, OUTPUT);
 pinMode(28, OUTPUT);
 pinMode(29, OUTPUT);
 pinMode(30, OUTPUT);
 pinMode(31, OUTPUT);
 digitalWrite(System_on,HIGH);

                               //RFID...
                               
  SPI.begin();                                                  // Init SPI bus
  mfrc522.PCD_Init();                                              // Init MFRC522 card
                            // Display...
                            
  lcd.begin();
  lcd.setCursor(4,1);
  lcd.print("!!!WELCOME!!!");
  delay(3000);
  
                          // Fingerprint...
                          
  while (!Serial);  
  //delay(100);                              // if you want delay adjust it
  finger.begin(57600);                      // set the data rate for the sensor serial port
  if (finger.verifyPassword()) 
  {
    Serial.println("Found fingerprint sensor!");
    lcd.setCursor(1,1);
    lcd.print("fingerprint Done!!!");
    //delay(2000);
  } 
  else 
  {
    Serial.println("Did not find fingerprint sensor :(");
    lcd.setCursor(1,1);
    lcd.print("fingerprint fail:(");
    while (1)
    { 
      delay(1); 
    }
  }
  lcd.clear();
  Serial.println(F("Ready to read the Rfid card"));          //shows in serial that it is ready to read
  lcd.setCursor(1,0);
  lcd.print("PLEASE");
  lcd.setCursor(1,1);
  lcd.print("PLACE THE CARD...");
  
}

uint8_t readnumber(void) 
{
  uint8_t num = 0;

  while (num == 0) 
  {
    while (! Serial.available());
    num = Serial.parseInt();                       //id input statement
  }
  Serial.print("\nid value: ");Serial.println(num);
  return num;
}

void loop()                                      // run over and over again
{

   MFRC522::MIFARE_Key key;                     // Prepare key - all keys are set to FFFFFFFFFFFFh at chip delivery from the factory.
   for (byte i = 0; i < 6; i++) key.keyByte[i] = 0xFF;
 
  byte block;                                  //some variables we need
  byte len;
  MFRC522::StatusCode status;
  if ( ! mfrc522.PICC_IsNewCardPresent())     // Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
  {
    return;
  }
  if ( ! mfrc522.PICC_ReadCardSerial())
  {
    return;
  }
  Serial.println(F("**Card Detected:**"));
  Serial.print(F("Name: "));
  byte buffer1[18];
  len = 18;
  byte buffer2[18];
  block = 1;
  status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, 1, &key, &(mfrc522.uid)); //line 834
  status = mfrc522.MIFARE_Read(block, buffer2, &len);
  
  String Rfid_value = "";                     //NAME
  for (uint8_t i = 0; i < 16; i++) 
  {
    Rfid_value += (char)buffer2[i];;         //Reading the data in RFID card 
  }
  Rfid_value.trim();
  Serial.print(Rfid_value);
  Serial.println(F("\n**End Reading**\n"));
  lcd.clear();
  int FingerPrintID;
  
  if(Rfid_value == "TN11 20210011429")
  {
    digitalWrite(Rfid_on,HIGH);
    lcd.setCursor(3,0);
    lcd.print("***HI KURAL***");
    Serial.println("***HI KURAL***");
    Serial.println("PLACE YOUR FINGERPRINT");
    lcd.setCursor(1,2);
    lcd.print("  PLACE YOUR");
    lcd.setCursor(1,3);
    lcd.print("     FINGERPRINT");
    FingerPrintID = getFingerprintIDez();
    if(FingerPrintID==4)
    {
      digitalWrite(Fingerprint_on,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("***HI KURAL***");
      Serial.println("FINGERPRINT DONE");
      lcd.setCursor(1,1);
      lcd.print(" FINGERPRINT DONE");
      Serial.println("CHECKING ALCOHOL CONTENT");
      lcd.setCursor(1,2);
      lcd.print("CHECKING ALCOHOL");
      lcd.setCursor(1,3);
      lcd.print("          CONTENT");
      delay(3000);
                                         
      float Alcoholcontent = GasSensor();        // calling gas sensor
      Serial.print("Alcoholcontent : ");Serial.println(Alcoholcontent); 
      if(Alcoholcontent > 1.99)
      {
      digitalWrite(Gas_off,HIGH);
      lcd.clear();
      lcd.setCursor(4,1);
      lcd.print("ALCOHOL FOUND");
      Serial.println("ALCOHOL FOUND");   
      lcd.setCursor(6,0);
      lcd.print("ALERT!!!");
      lcd.setCursor(1,3);
      lcd.print("YOU CAN'T DRIVE");
      while(1)
      {
      analogWrite(8,20);
      delay(1000);
      analogWrite(8,LOW);
      delay(700);
      }            
      }
      else
      {
      digitalWrite(Gas_on,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("ALCOHOL NOT FOUND");
      Serial.println("ALCOHOL NOT FOUND");
      lcd.setCursor(1,1);
      lcd.print("PLEASE WEAR THE");
      lcd.setCursor(1,2);
      lcd.print("    SEATBELT    ");
      lcd.setCursor(1,3);
      lcd.print("*NOW YOU CAN DRIVE*");
      Serial.println("PLEASE WEAR THE SEATBELT");
      Serial.println("*NOW YOU CAN DRIVE*");
       while(1)
        {
          //digitalWrite(Seatbelt_on,HIGH);               //Pin 49 output
          analogWrite(49, 200);                          //Ignition to start     
        } 
      } 
    }else
    {
      digitalWrite(Fingerprint_off,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("***HI KURAL***");
      Serial.print(" WRONG FINGERPRINT");
      Serial.print(" RESET ");
      lcd.setCursor(1,2);
      lcd.print(" WRONG FINGERPRINT");
      lcd.setCursor(1,3);
      lcd.print("   PRESS RESET"); 
      
      while(1)
      {
      analogWrite(8,20);
      delay(1000);
      analogWrite(8,LOW);
      delay(700);
      }      
     }
  }
  else if(Rfid_value == "TN91 20210003462")
  {
    digitalWrite(Rfid_on,HIGH);
    lcd.setCursor(1,0);
    lcd.print("***HI RAJESH***");
    Serial.println("***HI RAJESH***");
    Serial.println("PLACE YOUR FINGERPRINT");
    lcd.setCursor(1,2);
    lcd.print("  PLACE YOUR");
    lcd.setCursor(1,3);
    lcd.print("     FINGERPRINT");
    FingerPrintID = getFingerprintIDez();          //call to read fingerprint data  
    if(FingerPrintID == 1)
    {
      digitalWrite(Fingerprint_on,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("***HI RAJESH***");
      Serial.println("FINGERPRINT DONE");
      lcd.setCursor(1,1);
      lcd.print("FINGERPRINT DONE");
      Serial.println("CHECKING ALCOHOL CONTENT");
      lcd.setCursor(1,2);
      lcd.print("CHECKING ALCOHOL");
      lcd.setCursor(1,3);
      lcd.print("          CONTENT");
      delay(3000);
      //*********************************************************************************************************************************
      float Alcoholcontent = GasSensor();                              // gas sensor
      Serial.print("Alcoholcontent : ");Serial.println(Alcoholcontent); 
      if(Alcoholcontent > 1.99)
      {
      digitalWrite(Gas_off,HIGH);
      lcd.clear();
      lcd.setCursor(4,1);
      lcd.print("ALCOHOL FOUND");
      Serial.println("ALCOHOL FOUND");   
      lcd.setCursor(6,0);
      lcd.print("ALERT!!!");
      lcd.setCursor(1,3);
      lcd.print("YOU CAN'T DRIVE");
      while(1)
      {
      analogWrite(8,20);
      delay(1000);
      analogWrite(8,LOW);
      delay(700);
      } 
                 
      }
      else
      {
      digitalWrite(Gas_on,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("ALCOHOL NOT FOUND");
      Serial.println("ALCOHOL NOT FOUND");
      lcd.setCursor(1,1);
      lcd.print("PLEASE WEAR THE");
      lcd.setCursor(1,2);
      lcd.print("    SEATBELT    ");
      lcd.setCursor(1,3);
      lcd.print("*NOW YOU CAN DRIVE*");
      Serial.println("PLEASE WEAR THE SEATBELT");
      Serial.println("*NOW YOU CAN DRIVE*");
      while(1)
        {
          //digitalWrite(Seatbelt_on,HIGH);
           analogWrite(49, 200);                       //  pin 49 output      
        } 
      }
    }
    else
    {
      digitalWrite(Fingerprint_off,HIGH);
      lcd.clear();
      lcd.setCursor(1,0);
      lcd.print("***HI RAJESH***");
      Serial.print(" WRONG FINGERPRINT");
      Serial.print(" RESET ");
      lcd.setCursor(1,2);
      lcd.print(" WRONG FINGERPRINT");
      lcd.setCursor(1,3);
      lcd.print("   PRESS RESET"); 
      while(1)
      {
      analogWrite(8,20);
      delay(1000);
      analogWrite(8,LOW);
      delay(700);
      }      
      }
   }
else
 {
  digitalWrite(Rfid_off,HIGH);
  Serial.println("UNAUTHORIZED PERSON");
  lcd.clear();
  lcd.setCursor(1,2);
  lcd.print("UNAUTHORIZED PERSON");
  lcd.setCursor(6,0);
  lcd.print("ALERT!!!");
  digitalWrite(System_on,LOW);
  digitalWrite(System_off,HIGH);
  while(1)
  {
  analogWrite(8,20);
  delay(1000);
  analogWrite(8,LOW);
  delay(150);
  }
 }

   //*******************************************************************************************************************************************
}

int getFingerprintIDez()                   //Read the fingerprint data
{
  while(1)
  {
  uint8_t p = finger.getImage();
  //if (p != FINGERPRINT_OK) Serial.println("condition 1"); return -1;
  p = finger.image2Tz();
  //if (p != FINGERPRINT_OK) Serial.println("condition 2"); return -1;
  p = finger.fingerFastSearch();//if (p != FINGERPRINT_OK) Serial.println("condition 3");// return -1;
  if(finger.fingerID>=1)// found a match!
  {
  Serial.print("Found fingerprint ID = "); Serial.print(finger.fingerID); 
  Serial.println();
  return finger.fingerID; 
  }
}
}
//*************************************************************************
int GasSensor()                                //Gas sensor 
{
  pinMode(sensor, INPUT);
  float adcValue=0;
  for(int i=0;i<10;i++)
  {
    adcValue+= analogRead(sensor);                   //Reading alcohol content in gas  
    delay(10);
  }
    float v= (adcValue/10) * (5.0/1024.0);   
    float mgL= 0.67 * v;
    Serial.print("BAC:");
    Serial.print(mgL); Serial.println(" mg/L ");
    return mgL;                                     //Send the value to 'Alcoholcontent'                     
}
