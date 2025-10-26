#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int buttonTTest = 2;
const int buttonSEMO = 3;
const int buttonSlalom = 4;
const int trig = 5;
const int echo = 6;
float dist;
long dur;

enum Mode { NONE, T_TEST, SEMO_TEST, SLALOM_TEST };
Mode currentMode = NONE;

bool isTiming = false;
unsigned long startTime = 0;
unsigned long elapsedTime = 0;

void setup() {
  pinMode(buttonTTest, INPUT);
  pinMode(buttonSEMO, INPUT);
  pinMode(buttonSlalom, INPUT);
  pinMode(trig,OUTPUT);
  pinMode(echo,INPUT);

  lcd.begin();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("choose mode");
  delay(1000);
  lcd.setCursor(0, 1);
  lcd.print("T | S | SL");

  Serial.begin(9600);
  
}

void loop() {
  digitalWrite(trig,LOW);
  delayMicroseconds(5);
  /*digitalWrite(trig,HIGH);
  delayMicroseconds(5);
  digitalWrite(trig,LOW);
  dur = pulseIn(echo,HIGH);
  dist = microsecondsToCentimeters(dur);*/
  
  //if(dist > 0){
    if (digitalRead(buttonTTest) == HIGH) {
    setMode(T_TEST);
  } else if (digitalRead(buttonSEMO) == HIGH) {
    setMode(SEMO_TEST);
  } else if (digitalRead(buttonSlalom) == HIGH) {
    setMode(SLALOM_TEST);
  }

  if (currentMode != NONE) {
    lcd.setCursor(0, 1);
    lcd.print("Start/Stop");

    if (digitalRead(buttonTTest) == HIGH || digitalRead(buttonSEMO) == HIGH || digitalRead(buttonSlalom) == HIGH) {
      delay(200); // ป้องกันการกดซ้ำเร็วเกินไป
      if (!isTiming) {
        startTime = millis();
        isTiming = true;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Count...");
      } else {
        elapsedTime = millis() - startTime;
        isTiming = false;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("time:");
        lcd.setCursor(0, 1);
        lcd.print(elapsedTime / 1000.0, 2);
        lcd.print(" sec");
        delay(3000);
        resetState();
      }
    }
  }
//}
/*else{
  lcd.clear();
  lcd.setCursor(0,1);
  lcd.print("Setup yourself");
}*/
}

void setMode(Mode mode) {
  currentMode = mode;
  lcd.clear();
  lcd.setCursor(0, 0);
  switch (mode) {
    case T_TEST:
      lcd.print("Mode: T-test");
      break;
    case SEMO_TEST:
      lcd.print("Mode: SEMO");
      break;
    case SLALOM_TEST:
      lcd.print("Mode: Slalom");
      break;
    default:
      break;
  }
  delay(1000);
}

void resetState() {
  currentMode = NONE;
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("choose mode");
  lcd.setCursor(0, 1);
  lcd.print("T | S | SL");
}
long microsecondsToCentimeters(long microseconds){
  return microseconds / 29 / 2;
}
