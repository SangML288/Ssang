
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

int PinRST = 9;
int PinSDA = 10;
int UID[4], i;
MFRC522 My_mfRC522(PinSDA, PinRST);

void Read_Sensor();
void rotate1();

int ID1[4] = { 34, 38, 171, 34 };
int ID2[4] = { 179, 7, 93, 29 };
int ID3[4] = { 1, 120, 195, 28 };
int ID4[4] = { 16, 239, 192, 81 };
int ID5[4] = { 163, 238, 240, 29 };
// int ID6[4] = {211, 131, 90, 19};

int total_slot = 4;

Servo myservo;
int pos = 0;

#define ir_enter 7

#define ir_car1 5
#define ir_car2 4
#define ir_car3 3
#define ir_car4 2

#define COI 8
int S1 = 0, S2 = 0, S3 = 0, S4 = 0, S5 = 0, S6 = 0;
int C1 = 0, C2 = 0, C3 = 0, C4 = 0, C5 = 0, C6 = 0;
int flag1 = 0, flag2 = 0;
int Slot;

void setup() {

  myservo.attach(6);  // attaches the servo on pin 9 to the servo object
  Serial.begin(9600);
  //Khởi tạo RFID
  SPI.begin();
  myservo.write(90);
  My_mfRC522.PCD_Init();

  pinMode(ir_car1, INPUT);
  pinMode(ir_car2, INPUT);
  pinMode(ir_car3, INPUT);
  pinMode(ir_car4, INPUT);

  pinMode(ir_enter, INPUT);
  pinMode(COI, OUTPUT);
  SPEAK(5, 50);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(1, 0);
  lcd.print(" Car Parking ");
  lcd.setCursor(0, 1);
  lcd.print("SVTH: Sang-Nhut");
  delay(2000);
  lcd.clear();
}
int identification() {
  if (!My_mfRC522.PICC_IsNewCardPresent())  // Không có thẻ
    return 5;

  if (!My_mfRC522.PICC_ReadCardSerial())  //Thẻ không hợp lệ
    return 6;                             // Lỗi đọc thẻ

  for (byte i = 0; i < My_mfRC522.uid.size; i++) {
    Serial.print(My_mfRC522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    UID[i] = My_mfRC522.uid.uidByte[i];
    Serial.print(UID[i]);
  }

  if (UID[0] == ID1[0] && UID[1] == ID1[1] && UID[2] == ID1[2] && UID[3] == ID1[3])
    return 1;
  else if (UID[0] == ID2[0] && UID[1] == ID2[1] && UID[2] == ID2[2] && UID[3] == ID2[3])
    return 2;
  else if (UID[0] == ID3[0] && UID[1] == ID3[1] && UID[2] == ID3[2] && UID[3] == ID3[3])
    return 3;
  else if (UID[0] == ID4[0] && UID[1] == ID4[1] && UID[2] == ID4[2] && UID[3] == ID4[3])
    return 4;
  else if(UID[0] != ID1[0] || UID[1] != ID1[1] || UID[2] != ID1[2] || UID[3] != ID1[3] || 
          UID[0] != ID2[0] || UID[1] != ID2[1] || UID[2] != ID2[2] || UID[3] != ID2[3] || 
          UID[0] != ID3[0] || UID[1] != ID3[1] || UID[2] != ID3[2] || UID[3] != ID3[3] || 
          UID[0] != ID4[0] || UID[1] != ID4[1] || UID[2] != ID4[2] || UID[3] != ID4[3]) {
    return 0;
  }

  My_mfRC522.PICC_HaltA();
  My_mfRC522.PCD_StopCrypto1();
}
void loop() {
  Read_Sensor();
  Slot = total_slot - S1 - S2 - S3 - S4;
  if (Slot == 0) {
    lcd.setCursor(0, 0);
    lcd.print(" BAI DO DA DAY");
  } else {
    lcd.setCursor(0, 0);
    lcd.print(" BAI DO TRONG");
  }
  lcd.setCursor(0, 1);
  lcd.print("S1");
  lcd.setCursor(4, 1);
  lcd.print("S2");
  lcd.setCursor(8, 1);
  lcd.print("S3");
  lcd.setCursor(12, 1);
  lcd.print("S4");
while (digitalRead(ir_enter) == 0 && Slot != 0) {
    lcd.clear();
    while (digitalRead(ir_enter) == 0 && Slot != 0) {
      lcd.setCursor(0, 0);
      lcd.print(" MOI QUET THE!");
      int accessStatus = identification();  // Lưu trạng thái thẻ vào biến accessStatus
      if (accessStatus == 1) {              //Thẻ hợp lệ
        SPEAK(2, 100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("  THE HOP LE");
        if (C1 == 0) {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI VAO!");
        } else {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI RA!");
        }
        rotate1();
        delay(1000);
        lcd.clear();
        C1 = !C1;
      } else if (accessStatus == 2) {  //Thẻ hợp lệ
        SPEAK(2, 100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("  THE HOP LE");
        if (C2 == 0) {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI VAO!");
        } else {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI RA!");
        }
        rotate1();
        delay(1000);
        lcd.clear();
        C2 = !C2;
      } else if (accessStatus == 3) {  //Thẻ hợp lệ
        SPEAK(2, 100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("  THE HOP LE");
        if (C3 == 0) {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI VAO!");
        } else {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI RA!");
        }
        rotate1();
        delay(1000);
        lcd.clear();
        C3 = !C3;
      } else if (accessStatus == 4) {  //Thẻ hợp lệ
        SPEAK(2, 100);
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("  THE HOP LE");
        if (C4 == 0) {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI VAO!");
        } else {
          lcd.setCursor(0, 1);
          lcd.print("  XIN MOI RA!");
        }
        rotate1();
        delay(1000);
        lcd.clear();
        C4 = !C4;
      } else if (accessStatus == 6) {  //Thẻ hợp lệ
        lcd.setCursor(0, 0);
        lcd.print("  KHONG HOP LE!");
        SPEAK(1, 500);
        delay(2000);
        lcd.clear();
      }
    }
  }


  My_mfRC522.PICC_HaltA();
  My_mfRC522.PCD_StopCrypto1();
}


void Read_Sensor() {
  if (digitalRead(ir_car1) == 0) {
    S1 = 1;
    lcd.setCursor(2, 1);
    lcd.print("T");
  } else {
    S1 = 0;
    lcd.setCursor(2, 1);
    lcd.print("F");
  }
  if (digitalRead(ir_car2) == 0) {
    S2 = 1;
    lcd.setCursor(6, 1);
    lcd.print("T");
  } else {
    S2 = 0;
    lcd.setCursor(6, 1);
    lcd.print("F");
  }
  if (digitalRead(ir_car3) == 0) {
    S3 = 1;
    lcd.setCursor(10, 1);
    lcd.print("T");
  } else {
    S3 = 0;
    lcd.setCursor(10, 1);
    lcd.print("F");
  }
  if (digitalRead(ir_car4) == 0) {
    S4 = 1;
    lcd.setCursor(14, 1);
    lcd.print("T");
  } else {
    S4 = 0;
    lcd.setCursor(14, 1);
    lcd.print("F");
  }
}

void rotate1() {
