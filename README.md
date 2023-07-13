#include <Adafruit_Fingerprint.h>
#if (defined(_AVR) || defined(ESP8266)) && !defined(__AVR_ATmega2560_)
// For UNO and others without hardware serial, we must use software serial...
// pin #2 is IN from sensor (GREEN wire)
// pin #3 is OUT from arduino (WHITE wire)
// Set up the serial port to use softwareserial..
SoftwareSerial mySerial(2, 3);
#else
// On Leonardo/M0/etc, others with hardware serial, use hardware serial!
// #0 is green wire, #1 is white
#define mySerial Serial1
#endif
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);
uint8_t id;
void setup()
{
 Serial.begin(57600);
 while (!Serial); // For Yun/Leo/Micro/Zero/...
 delay(100);
 Serial.println("\n\nAdafruit Fingerprint sensor enrollment");
 // set the data rate for the sensor serial port
 finger.begin(57600);
 if (finger.verifyPassword()) {
 Serial.println("Found fingerprint sensor!");
 } else {
 Serial.println("Did not find fingerprint sensor :(");
 while (1) { delay(1); }
 }
Serial.println(F("Reading sensor parameters"));
 finger.getParameters();
 Serial.print(F("Status: 0x")); Serial.println(finger.status_reg, HEX);
 Serial.print(F("Sys ID: 0x")); Serial.println(finger.system_id, HEX);
 Serial.print(F("Capacity: ")); Serial.println(finger.capacity);
 Serial.print(F("Security level: ")); Serial.println(finger.security_level);
 Serial.print(F("Device address: ")); Serial.println(finger.device_addr, HEX);
 Serial.print(F("Packet len: ")); Serial.println(finger.packet_len);
 Serial.print(F("Baud rate: ")); Serial.println(finger.baud_rate);
}
uint8_t readnumber(void) {
 uint8_t num = 0;
 while (num == 0) {
 while (! Serial.available());
 num = Serial.parseInt();
 }
 return num;
}
void loop()// run over and over again
{
 Serial.println("Ready to enroll a fingerprint!");
 Serial.println("Please type in the ID # (from 1 to 127) you want to save this finger as...");
 id = readnumber();
 if (id == 0) {// ID #0 not allowed, try again!
return;
 }
 Serial.print("Enrolling ID #");
 Serial.println(id);
 while (! getFingerprintEnroll() );
}
uint8_t getFingerprintEnroll() {
 int p = -1;
 Serial.print("Waiting for valid finger to enroll as #"); Serial.println(id)
while (p != FINGERPRINT_OK) {
 p = finger.getImage();
 delay(1000);
 switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image taken");
 break;
 case FINGERPRINT_NOFINGER:
 Serial.println(".");
 break;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 break;
 case FINGERPRINT_IMAGEFAIL:
 Serial.println("Imaging error");
 break;
 default:
 Serial.println("Unknown error");
 break;
 }
 }
 // OK success!
 p = finger.image2Tz(1);
 switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image converted");
 break;
 case FINGERPRINT_IMAGEMESS:
 Serial.println("Image too messy");
 return p;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 return p;
 case FINGERPRINT_FEATUREFAIL:
 Serial.println("Could not find fingerprint features");
 return p;
 case FINGERPRINT_INVALIDIMAGE:
 Serial.println("Could not find fingerprint features");
return p;
 default:
 Serial.println("Unknown error");
 return p;
 }
 Serial.println("Remove finger");
 delay(2000);
 p = 0;
 while (p != FINGERPRINT_NOFINGER) {
 p = finger.getImage();
 }
 Serial.print("ID "); Serial.println(id);
 p = -1;
 Serial.println("Place same finger again");
 while (p != FINGERPRINT_OK) {
 p = finger.getImage();
 delay(1000);
 switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image taken");
 break;
 case FINGERPRINT_NOFINGER:
 Serial.print(".");
 break;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 break;
 case FINGERPRINT_IMAGEFAIL:
 Serial.println("Imaging error");
 break;
 default:
 Serial.println("Unknown error");
 break;
 }
 }
 // OK success!
 p = finger.image2Tz(2);
switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image converted");
 break;
 case FINGERPRINT_IMAGEMESS:
 Serial.println("Image too messy");
 return p;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 return p;
 case FINGERPRINT_FEATUREFAIL:
 Serial.println("Could not find fingerprint features");
 return p;
 case FINGERPRINT_INVALIDIMAGE:
 Serial.println("Could not find fingerprint features");
 return p;
 default:
 Serial.println("Unknown error");
 return p;
 }
 // OK converted!
 Serial.print("Creating model for #"); Serial.println(id);
 p = finger.createModel();
 if (p == FINGERPRINT_OK) {
 Serial.println("Prints matched!");
 } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
 Serial.println("Communication error");
 return p;
 } else if (p == FINGERPRINT_ENROLLMISMATCH) {
 Serial.println("Fingerprints did not match");
 return p;
 } else {
 Serial.println("Unknown error");
 return p;
 }
 Serial.print("ID "); Serial.println(id);
 p = finger.storeModel(id);
if (p == FINGERPRINT_OK) {
 Serial.println("Stored!");
 } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
 Serial.println("Communication error");
 return p;
 } else if (p == FINGERPRINT_BADLOCATION) {
 Serial.println("Could not store in that location");
 return p;
 } else if (p == FINGERPRINT_FLASHERR) {
 Serial.println("Error writing to flash");
 return p;
 } else {
 Serial.println("Unknown error");
 return p;
 }
 return true;
}
[5/20, 11:38 PM] Roshan Jain: #include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_Fingerprint.h>
LiquidCrystal_I2C lcd(0×27,16,2); // set the LCD address to 0×27 for a 16 chars and 2 line
display
int relay=8;
int buzzer=10;
SoftwareSerial mySerial(2, 3);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);
void setup()
{ pinMode(relay,OUTPUT);
 pinMode(buzzer,OUTPUT);
 digitalWrite(relay,HIGH);
 lcd.init(); // initialize the lcd
 lcd.init();
 // Print a message to the LCD.
 lcd.backlight();
 while (!Serial); // For Yun/Leo/Micro/Zero/...
 delay(100);
 Serial.println("\n\nAdafruit finger detect test");
finger.begin(57600);
 delay(5);
 if (finger.verifyPassword()) {
 Serial.println("Found fingerprint sensor!");
 } else {
 Serial.println("Did not find fingerprint sensor :(");
 while (1) { delay(1); }
}
Serial.println(F("Reading sensor parameters"));
 finger.getParameters();
 Serial.print(F("Status: 0x")); Serial.println(finger.status_reg, HEX);
 Serial.print(F("Sys ID: 0x")); Serial.println(finger.system_id, HEX);
 Serial.print(F("Capacity: ")); Serial.println(finger.capacity);
 Serial.print(F("Security level: ")); Serial.println(finger.security_level);
 Serial.print(F("Device address: ")); Serial.println(finger.device_addr, HEX);
 Serial.print(F("Packet len: ")); Serial.println(finger.packet_len);
 Serial.print(F("Baud rate: ")); Serial.println(finger.baud_rate);
 finger.getTemplateCount();
 if (finger.templateCount == 0) {
 Serial.print("Sensor doesn't contain any fingerprint data. Please run the 'enroll'
example.");
 }
 else {
 Serial.println("Waiting for valid finger...");
 Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println("
templates");
 }
}
void loop() // run over and over again
{ lcd.clear();
 lcd.setCursor(0, 0);
lcd.print("Fingerprint Door");
lcd.setCursor(0, 1);
lcd.print("lock ");
delay(3000);
 getFingerprintID();
 delay(50);
if (finger.confidence > 65)
{ lcd.clear();
lcd.setCursor(0, 0);
lcd.print(" Door Unlocked");
lcd.setCursor(0, 1);
lcd.print(" Welcome");
 digitalWrite(relay,LOW);
 delay(5000);
 finger.confidence=0;
}
 digitalWrite(relay,HIGH);
}
uint8_t getFingerprintID() {
 uint8_t p = finger.getImage();
 switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image taken");
 break;
 case FINGERPRINT_NOFINGER:
 Serial.println("No finger detected");
 return p;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 return p;
 case FINGERPRINT_IMAGEFAIL:
 Serial.println("Imaging error");
 return p;
 default:
 Serial.println("Unknown error");
 return p;
 }
 // OK success!
 p = finger.image2Tz();
switch (p) {
 case FINGERPRINT_OK:
 Serial.println("Image converted");
 break;
 case FINGERPRINT_IMAGEMESS:
 Serial.println("Image too messy");
 return p;
 case FINGERPRINT_PACKETRECIEVEERR:
 Serial.println("Communication error");
 return p;
 case FINGERPRINT_FEATUREFAIL:
 Serial.println("Could not find fingerprint features");
 return p;
 case FINGERPRINT_INVALIDIMAGE:
 Serial.println("Could not find fingerprint features");
 return p;
 default:
 Serial.println("Unknown error");
 return p;
 }
 // OK converted!
 p = finger.fingerSearch();
 if (p == FINGERPRINT_OK) {
 Serial.println("Found a print match!");
 } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
 Serial.println("Communication error");
 return p;
 } else if (p == FINGERPRINT_NOTFOUND) {
 digitalWrite(buzzer,HIGH);
 lcd.clear();
lcd.setCursor(0, 0);
lcd.print("Sorry");
lcd.setCursor(0, 1);
lcd.print("Wrong finger");
 delay(3000);
 digitalWrite(buzzer,LOW);
 Serial.println("Did not find a match");
 return p;
 } else {
Serial.println("Unknown error");
 return p;
 }
 // found a match!
 Serial.print("Found ID #"); Serial.print(finger.fingerID);
 Serial.print(" with confidence of "); Serial.println(finger.confidence);
 return finger.fingerID;
}
// returns -1 if failed, otherwise returns ID #
int getFingerprintIDez() {
 uint8_t p = finger.getImage();
 if (p != FINGERPRINT_OK) return -1;
 p = finger.image2Tz();
 if (p != FINGERPRINT_OK) return -1;
 p = finger.fingerFastSearch();
 if (p != FINGERPRINT_OK) return -1;
 // found a match!
 Serial.print("Found ID #"); Serial.print(finger.fingerID);
 Serial.print(" with confidence of "); Serial.println(finger.confidence);
 return finger.finger ID;
}
