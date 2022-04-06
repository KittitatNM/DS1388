# ปรับแก้ RTClib สำหรับการใช้งาน DS1388

เนื่องจากการใช้งาน RTClib จาก [adafruit/RTClib](https://github.com/adafruit/RTClib) แล้วประสบปัญหาการใช้งานกับ DS1388 จึงได้มีการปรับปรุงเพิ่มเติมเพื่อให้ใช้งานได้

## RTClib.h

ปรับแก้ไฟล์ RTClib.h เพื่อสร้าง Class ในการกำหนดโครงสร้างของ DS1388 โดยการเพิ่ม Code ด้านล่างลงไป

```C
class RTC_DS1388 {
  public:
    bool begin();
    void adjust(const DateTime &dt);
    DateTime now();
};
```

## RTClib.cpp

ปรับแก้ไฟล์ RTClib.cpp เพื่อการปรับแต่งการทำงานของ Class จากไฟล์ RTClib.h เพื่อให้ Class DS1388 ใช้งานได้จริง โดยการเพิ่ม Code ด้านล่างลงไป

```C
bool RTC_DS1388::begin() {
  Wire.begin();
  Wire.beginTransmission(DS1307_ADDRESS);
  if (Wire.endTransmission() == 0)
    return true;
  return false;
}

void RTC_DS1388::adjust(const DateTime &dt) {
  Wire.beginTransmission(DS1307_ADDRESS);
  Wire._I2C_WRITE((byte)0); // start at location 0
  Wire._I2C_WRITE((byte)0);
  Wire._I2C_WRITE(bin2bcd(dt.second()));
  Wire._I2C_WRITE(bin2bcd(dt.minute()));
  Wire._I2C_WRITE(bin2bcd(dt.hour()));
  Wire._I2C_WRITE(bin2bcd(0));
  Wire._I2C_WRITE(bin2bcd(dt.day()));
  Wire._I2C_WRITE(bin2bcd(dt.month()));
  Wire._I2C_WRITE(bin2bcd(dt.year() - 2000U));
  Wire.endTransmission();
}

DateTime RTC_DS1388::now() {
  Wire.beginTransmission(DS1307_ADDRESS);
  Wire._I2C_WRITE((byte)0);
  Wire.endTransmission();

  Wire.requestFrom(DS1307_ADDRESS, 8);
  uint8_t ms = bcd2bin(Wire._I2C_READ());
  uint8_t ss = bcd2bin(Wire._I2C_READ() & 0x7F);
  uint8_t mm = bcd2bin(Wire._I2C_READ());
  uint8_t hh = bcd2bin(Wire._I2C_READ());
  Wire._I2C_READ();
  uint8_t d = bcd2bin(Wire._I2C_READ());
  uint8_t m = bcd2bin(Wire._I2C_READ());
  uint16_t y = bcd2bin(Wire._I2C_READ()) + 2000U;

  return DateTime(y, m, d, hh, mm, ss);
}
```

## ตัวอย่างการใช้งาน

```C
RTC_DS1388 rtc;


void setup() {
  Serial.begin(115200);
  rtc.begin();

  //  rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); ใช้สำหรับตั้งเวลา
}

void loop() {
  DateTime now = rtc.now();

  Serial.print(now.year(), DEC);
  Serial.print('/');
  Serial.print(now.month(), DEC);
  Serial.print('/');
  Serial.print(now.day(), DEC);
  Serial.print(" (");
  Serial.print(daysOfTheWeek[now.dayOfTheWeek()]);
  Serial.print(") ");
  Serial.print(now.hour(), DEC);
  Serial.print(':');
  Serial.print(now.minute(), DEC);
  Serial.print(':');
  Serial.print(now.second(), DEC);
  Serial.println();

  Serial.print(" since midnight 1/1/1970 = ");
  Serial.print(now.unixtime());
  Serial.print("s = ");
  Serial.print(now.unixtime() / 86400L);
  Serial.println("d");

  // calculate a date which is 7 days, 12 hours, 30 minutes, and 6 seconds into the future
  DateTime future (now + TimeSpan(7, 12, 30, 6));

  Serial.print(" now + 7d + 12h + 30m + 6s: ");
  Serial.print(future.year(), DEC);
  Serial.print('/');
  Serial.print(future.month(), DEC);
  Serial.print('/');
  Serial.print(future.day(), DEC);
  Serial.print(' ');
  Serial.print(future.hour(), DEC);
  Serial.print(':');
  Serial.print(future.minute(), DEC);
  Serial.print(':');
  Serial.print(future.second(), DEC);
  Serial.println();

  Serial.println();
  delay(3000);
}
```
