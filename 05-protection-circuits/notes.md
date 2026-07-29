# 🔌 Comprehensive ESP32 Notes - Phase 5: Sensors & Actuators
### For Network Security Engineers and Red Teamers

---

> **Prerequisite:** Phases 1 through 4 completed.
> **Goal of this phase:** full mastery of the sensors and actuators common in IoT — both from a practical setup perspective, and from the perspective of a Red Teamer who wants to know what hardware target devices have and how it works.

---

## 📋 Table of Contents

- [Chapter 5.1 - Environmental Sensors](#chapter-51---environmental-sensors)
- [Chapter 5.2 - Motion & Position Sensors](#chapter-52---motion--position-sensors)
- [Chapter 5.3 - Visual & Optical Sensors](#chapter-53---visual--optical-sensors)
- [Chapter 5.4 - Communicating with External Devices](#chapter-54---communicating-with-external-devices)
- [Chapter 5.5 - Actuators](#chapter-55---actuators)

---

---

# Chapter 5.1 - Environmental Sensors

---

## Introduction: What is a Sensor?

A sensor is a device that converts a **physical quantity** (temperature, pressure, light, ...) into an **electrical signal** that a microcontroller can read.

```
Physical world          Sensor           ESP32
   Temp 25°C    →    [DHT22]    →   value 25.5
   Pressure 1atm →   [BMP280]   →   value 101325
   Light on      →    [LDR]     →   value 3800
```

**From a Red Team perspective:**
The sensors on a target IoT device reveal **what data the device is collecting**. Knowing the sensor type → understanding the data flow → finding the attack surface.

---

## 1. DHT11 / DHT22 - Temperature and Humidity

### DHT11 vs. DHT22

```
DHT11:
  Temperature: 0-50°C  ± 2°C
  Humidity: 20-80%  ± 5%
  Rate:   1 reading per second
  Price:  cheaper
  Protocol: proprietary 1-wire

DHT22 (AM2302):
  Temperature: -40 to +80°C  ± 0.5°C
  Humidity: 0-100%  ± 2-5%
  Rate:   0.5 readings per second (once every 2 seconds)
  Price:  slightly more expensive
  Protocol: same as DHT11

Recommendation: DHT22 is better for most projects
```

### Wiring

```
DHT22          ESP32
  VCC  ──────── 3.3V or 5V
  DATA ──────── GPIO4 (+ 10kΩ resistor to VCC)
  NC   ──────── (unconnected)
  GND  ──────── GND

Important note: the 10kΩ pull-up resistor is mandatory!
Without this resistor, data can't be read.
```

### Code - DHT library

```cpp
#include <DHT.h>

#define DHT_PIN  4       // GPIO4
#define DHT_TYPE DHT22   // or DHT11

DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
    Serial.begin(115200);
    dht.begin();
    Serial.println("DHT22 initialized");
}

void loop() {
    // Reading (slow - takes about 250ms)
    float humidity    = dht.readHumidity();
    float temperature = dht.readTemperature();      // Celsius
    float tempF       = dht.readTemperature(true);  // Fahrenheit

    // Error check
    if (isnan(humidity) || isnan(temperature)) {
        Serial.println("DHT read failed!");
        delay(2000);
        return;
    }

    // Calculate Heat Index (real-feel temperature)
    float heatIndex = dht.computeHeatIndex(temperature, humidity, false);

    Serial.printf("Temperature: %.1f°C (%.1f°F)\n",
                   temperature, tempF);
    Serial.printf("Humidity: %.1f%%\n", humidity);
    Serial.printf("Heat Index: %.1f°C\n", heatIndex);

    delay(2000);  // DHT22 at most once every 2 seconds
}
```

### The DHT protocol - how it works

```
DHT's proprietary 1-wire protocol:

ESP32 → DHT:  hold the pin LOW for 18ms (Start Signal)
ESP32 → DHT:  set the pin HIGH (20-40µs)
DHT → ESP32:  80µs LOW + 80µs HIGH (Response)
DHT → ESP32:  sends 40 bits of data

Each bit:
  0: 50µs LOW + 26-28µs HIGH
  1: 50µs LOW + 70µs HIGH

40 bits = 5 bytes:
  Byte 0: RH integer part
  Byte 1: RH decimal part
  Byte 2: T integer part
  Byte 3: T decimal part
  Byte 4: Checksum = sum of first 4 bytes
```

### Code without a library (low-level)

```cpp
// Reading the DHT22 with bit-banging
uint8_t dht_data[5] = {0};

bool dht_read_raw(uint8_t pin) {
    // Start signal
    pinMode(pin, OUTPUT);
    digitalWrite(pin, LOW);
    delay(18);  // 18ms
    digitalWrite(pin, HIGH);
    delayMicroseconds(30);
    pinMode(pin, INPUT_PULLUP);

    // Response
    delayMicroseconds(80);
    if (digitalRead(pin) != HIGH) return false;
    delayMicroseconds(80);

    // Read 40 bits
    for (int i = 0; i < 40; i++) {
        while (digitalRead(pin) == LOW);  // wait for HIGH

        delayMicroseconds(35);  // sampling point

        if (digitalRead(pin) == HIGH)
            dht_data[i/8] |= (1 << (7 - i%8));

        while (digitalRead(pin) == HIGH);  // wait for LOW
    }

    // checksum
    return dht_data[4] == (dht_data[0] + dht_data[1] +
                            dht_data[2] + dht_data[3]);
}
```

---

## 2. BME280 / BMP280

### BME280 vs. BMP280

```
BMP280:
  Temperature + pressure
  Good for: altimeters, weather stations

BME280:
  Temperature + pressure + humidity
  Good for: full weather stations

Both support I2C or SPI
I2C address: 0x76 (SDO=GND) or 0x77 (SDO=VCC)
```

### I2C wiring

```
BME280          ESP32
  VCC  ──────── 3.3V (important: not 5V!)
  GND  ──────── GND
  SDA  ──────── GPIO21
  SCL  ──────── GPIO22
  SDO  ──────── GND (address 0x76) or VCC (address 0x77)
  CSB  ──────── VCC (for I2C mode)
```

### Code - Adafruit BME280 library

```cpp
#include <Wire.h>
#include <Adafruit_BME280.h>

Adafruit_BME280 bme;

void setup() {
    Serial.begin(115200);
    Wire.begin(21, 22);  // SDA, SCL

    if (!bme.begin(0x76)) {
        Serial.println("BME280 not found!");
        while (1);
    }

    Serial.println("BME280 initialized");

    // Configure operating mode
    // WEATHER_MONITORING: high accuracy, low consumption
    bme.setSampling(
        Adafruit_BME280::MODE_NORMAL,     // operating mode
        Adafruit_BME280::SAMPLING_X16,    // temperature oversampling
        Adafruit_BME280::SAMPLING_X16,    // pressure oversampling
        Adafruit_BME280::SAMPLING_X16,    // humidity oversampling
        Adafruit_BME280::FILTER_X16,      // noise filter
        Adafruit_BME280::STANDBY_MS_500   // standby time
    );
}

void loop() {
    float temperature = bme.readTemperature();      // °C
    float pressure    = bme.readPressure() / 100.0; // hPa
    float humidity    = bme.readHumidity();         // %
    float altitude    = bme.readAltitude(1013.25);  // meters (with sea-level pressure)

    Serial.printf("Temp: %.2f°C\n", temperature);
    Serial.printf("Pressure: %.2f hPa\n", pressure);
    Serial.printf("Humidity: %.2f%%\n", humidity);
    Serial.printf("Altitude: %.2f m\n", altitude);

    delay(2000);
}
```

### Reading via SPI

```cpp
#include <SPI.h>
#include <Adafruit_BME280.h>

#define BME_CS   5   // Chip Select
#define BME_MOSI 23
#define BME_MISO 19
#define BME_SCK  18

Adafruit_BME280 bme(BME_CS, BME_MOSI, BME_MISO, BME_SCK);
// or with Hardware SPI:
// Adafruit_BME280 bme(BME_CS);

void setup() {
    if (!bme.begin()) {
        Serial.println("BME280 SPI failed!");
    }
}
```

### Precise altitude calculation

```cpp
// Barometric formula:
// altitude = 44330 * (1 - (pressure/seaLevelPressure)^(1/5.255))

float calculateAltitude(float pressure_hPa) {
    const float SEA_LEVEL = 1013.25;  // hPa
    return 44330.0 * (1.0 - pow(pressure_hPa / SEA_LEVEL, 0.1903));
}

// For greater accuracy: you need the local sea-level pressure
// You can get this from the OpenWeatherMap API
```

---

## 3. DS18B20 - Precise Temperature with 1-Wire

### DS18B20 Specs

```
Accuracy: ±0.5°C over -10 to +85°C
Range: -55 to +125°C
Resolution: 9 to 12 bits (adjustable)
Protocol: Dallas 1-Wire
Address: unique 64-bit (ROM Code)
Power: Parasitic (from the DATA line) or VCC

Key advantage: you can have several DS18B20s on a single wire!
```

### Wiring

```
DS18B20        ESP32
  VDD ──────── 3.3V
  GND ──────── GND
  DQ  ──────── GPIO4 (+ 4.7kΩ resistor to VCC)

For several sensors:
  all DQ pins to one pin + one 4.7kΩ resistor to VCC
```

### Code - DallasTemperature library

```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS 4

OneWire         oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

void setup() {
    Serial.begin(115200);
    sensors.begin();

    // Number of sensors found
    int count = sensors.getDeviceCount();
    Serial.printf("Found %d DS18B20 sensor(s)\n", count);

    // Show each sensor's address
    DeviceAddress addr;
    for (int i = 0; i < count; i++) {
        if (sensors.getAddress(addr, i)) {
            Serial.printf("Sensor %d address: ", i);
            for (int j = 0; j < 8; j++)
                Serial.printf("%02X ", addr[j]);
            Serial.println();
        }
    }

    // Set resolution
    sensors.setResolution(12);  // 12-bit: 0.0625°C accuracy, 750ms delay
    // 9-bit: 0.5°C accuracy,  94ms delay
    // 10-bit: 0.25°C accuracy, 188ms delay
    // 11-bit: 0.125°C accuracy, 375ms delay
}

void loop() {
    // Request conversion (all sensors at once)
    sensors.requestTemperatures();

    // Read the first sensor
    float temp0 = sensors.getTempCByIndex(0);
    Serial.printf("Sensor 0: %.4f°C\n", temp0);

    // Read by a specific address
    DeviceAddress myAddr = {0x28, 0xAA, 0xBB, 0xCC,
                            0xDD, 0xEE, 0xFF, 0x01};
    float tempAddr = sensors.getTempC(myAddr);
    Serial.printf("Sensor by address: %.4f°C\n", tempAddr);

    delay(1000);
}
```

### The Dallas 1-Wire protocol

```
ROM Commands:
  0x33: READ ROM      → read the 64-bit address (single sensor only)
  0x55: MATCH ROM     → select a sensor by a specific address
  0xCC: SKIP ROM      → broadcast to all
  0xF0: SEARCH ROM    → find all sensors

Function Commands:
  0x44: CONVERT T     → start a temperature reading
  0xBE: READ SCRATCHPAD → read the 9-byte result
  0x4E: WRITE SCRATCHPAD → write settings
```

---

## 4. MQ Series Gas Sensors

### Types of MQ sensors

```
MQ-2:  smoke, methane (CH4), propane, hydrogen
MQ-3:  alcohol, ethanol, alcohol vapor
MQ-4:  methane, natural gas
MQ-5:  LPG, methane, propane
MQ-6:  LPG, propane, butane
MQ-7:  carbon monoxide (CO)
MQ-8:  hydrogen
MQ-9:  CO, combustible gases
MQ-135: ammonia, benzene, CO2, smoke
MQ-136: hydrogen sulfide gas (H2S)
```

### How MQ sensors work

```
An MQ sensor uses a sensitive ceramic element coated with SnO2:

Clean air:
  SnO2 has high resistance → low output voltage

Polluted air:
  target gas → reacts with SnO2 → resistance drops → output voltage rises

Output:
  AOUT pin: analog signal (0-5V) → connect to an ADC
  DOUT pin: digital signal (HIGH/LOW) → threshold set via a trimmer
```

### Wiring

```
MQ Sensor         ESP32
  VCC    ──────── 5V (important: must be 5V!)
  GND    ──────── GND
  AOUT   ──────── GPIO34 (ADC1_CH6)
  DOUT   ──────── GPIO35 (optional)

Problem: the MQ needs 5V but the ESP32's ADC only handles 3.3V
Solution: a voltage divider on AOUT
  AOUT ─── 10kΩ ─── GPIO34
                └── 20kΩ ─── GND
  Divided voltage: 5 × (20/(10+20)) = 3.33V ✅
```

### Code

```cpp
#define MQ2_AOUT_PIN 34
#define MQ2_DOUT_PIN 35

// Ro: the sensor's resistance in clean air
// needs to be calibrated once (usually ~9.83kΩ for the MQ-2)
float Ro = 9.83;

// Read Rs (current resistance)
float readRs(int pin) {
    int raw = analogRead(pin);
    float voltage = raw * 3.3 / 4095.0;
    // voltage-divider circuit with RL=10kΩ
    if (voltage < 0.01) return 1000.0;  // avoid dividing by zero
    float Rs = (3.3 - voltage) / voltage * 10.0;  // kΩ
    return Rs;
}

// Calculate the Rs/Ro ratio
float readRatio(int pin) {
    return readRs(pin) / Ro;
}

// Estimate concentration from the datasheet curve (PPM)
// this formula is derived from the MQ-2 datasheet curve
float readPPM_LPG(float ratio) {
    return 1000.65 * pow(ratio, -2.506);
}

float readPPM_CO(float ratio) {
    return 36.124 * pow(ratio, -3.118);
}

void setup() {
    Serial.begin(115200);
    analogSetAttenuation(ADC_11db);  // for 0-3.6V

    Serial.println("MQ-2 warming up (20 seconds)...");
    delay(20000);  // preheating is required!
    Serial.println("Ready");
}

void loop() {
    float ratio = readRatio(MQ2_AOUT_PIN);
    float lpg_ppm = readPPM_LPG(ratio);
    float co_ppm  = readPPM_CO(ratio);
    bool alarm    = digitalRead(MQ2_DOUT_PIN) == LOW;

    Serial.printf("Rs/Ro: %.2f | LPG: %.1f ppm | CO: %.1f ppm | Alarm: %s\n",
                   ratio, lpg_ppm, co_ppm, alarm ? "YES!" : "No");

    delay(1000);
}
```

### MQ calibration

```cpp
// Calibration in clean air
float calibrate_MQ(int pin, int samples = 50) {
    Serial.println("Calibrating MQ sensor in clean air...");
    delay(20000);  // preheating

    float sum = 0;
    for (int i = 0; i < samples; i++) {
        sum += readRs(pin);
        delay(500);
    }
    float Ro_cal = sum / samples;
    Serial.printf("Calibrated Ro: %.2f kΩ\n", Ro_cal);
    return Ro_cal;
}
```

---

---

# Chapter 5.2 - Motion & Position Sensors

---

## 1. MPU6050 - Accelerometer + Gyroscope

### What is the MPU6050?

The MPU6050 is an **IMU (Inertial Measurement Unit)** that combines two sensors in a single chip:

```
Accelerometer:
  measures acceleration on the X, Y, Z axes
  range: ±2g, ±4g, ±8g, ±16g
  use: angle detection, impact, motion, free-fall

Gyroscope:
  measures angular velocity on X, Y, Z
  range: ±250, ±500, ±1000, ±2000 deg/s
  use: rotation detection, stabilization

DMP (Digital Motion Processor):
  an internal processor for data fusion
  produces a quaternion (3D orientation)
```

### Wiring

```
MPU6050        ESP32
  VCC  ──────── 3.3V
  GND  ──────── GND
  SDA  ──────── GPIO21
  SCL  ──────── GPIO22
  INT  ──────── GPIO19 (optional - interrupt)
  AD0  ──────── GND (address 0x68) or VCC (address 0x69)
```

### Code - MPU6050 library

```cpp
#include <Wire.h>
#include <MPU6050.h>

MPU6050 mpu;

void setup() {
    Serial.begin(115200);
    Wire.begin(21, 22);

    mpu.initialize();

    if (!mpu.testConnection()) {
        Serial.println("MPU6050 connection failed!");
        while (1);
    }
    Serial.println("MPU6050 connected");

    // Set range
    mpu.setFullScaleAccelRange(MPU6050_ACCEL_FS_2);  // ±2g
    mpu.setFullScaleGyroRange(MPU6050_GYRO_FS_250);  // ±250°/s

    // Calibration offsets
    mpu.setXAccelOffset(-1698);
    mpu.setYAccelOffset(-1120);
    mpu.setZAccelOffset(1399);
    mpu.setXGyroOffset(56);
    mpu.setYGyroOffset(-12);
    mpu.setZGyroOffset(10);
}

void loop() {
    int16_t ax, ay, az;
    int16_t gx, gy, gz;

    // Read raw data
    mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);

    // Convert to physical units
    float accel_scale = 2.0 / 32768.0;    // for ±2g
    float gyro_scale  = 250.0 / 32768.0;  // for ±250°/s

    float Ax = ax * accel_scale;  // g
    float Ay = ay * accel_scale;
    float Az = az * accel_scale;

    float Gx = gx * gyro_scale;  // deg/s
    float Gy = gy * gyro_scale;
    float Gz = gz * gyro_scale;

    // The chip's internal temperature
    float temp = mpu.getTemperature() / 340.0 + 36.53;

    Serial.printf("Accel: X=%.2fg Y=%.2fg Z=%.2fg\n", Ax, Ay, Az);
    Serial.printf("Gyro:  X=%.1f Y=%.1f Z=%.1f deg/s\n", Gx, Gy, Gz);
    Serial.printf("Temp: %.1f°C\n", temp);

    delay(100);
}
```

### Calculating tilt angle

```cpp
#include <math.h>

// Calculate angle from the accelerometer
// (accurate for a stationary orientation, not fast motion)
void calculateAngles(float ax, float ay, float az,
                     float* roll, float* pitch) {
    *roll  = atan2(ay, az) * 180.0 / PI;
    *pitch = atan2(-ax, sqrt(ay*ay + az*az)) * 180.0 / PI;
}

// Complementary Filter: combining Accel + Gyro
// for a better result
float compAngleX = 0, compAngleY = 0;
unsigned long lastTime = 0;

void complementaryFilter(float ax, float ay, float az,
                          float gx, float gy) {
    float dt = (millis() - lastTime) / 1000.0;
    lastTime = millis();

    float accel_roll, accel_pitch;
    calculateAngles(ax, ay, az, &accel_roll, &accel_pitch);

    // 0.98 from the gyro (fast but drifts)
    // 0.02 from the accel (slow but no drift)
    compAngleX = 0.98 * (compAngleX + gx * dt) + 0.02 * accel_roll;
    compAngleY = 0.98 * (compAngleY + gy * dt) + 0.02 * accel_pitch;
}
```

### Impact and free-fall detection

```cpp
void setupMotionDetection() {
    // Impact detection (Motion Interrupt)
    mpu.setMotionDetectionThreshold(10);  // threshold
    mpu.setMotionDetectionDuration(40);   // duration (ms)
    mpu.setIntMotionEnabled(true);

    // Free-fall detection
    mpu.setFreeFallDetectionThreshold(17);
    mpu.setFreeFallDetectionDuration(100);
    mpu.setIntFreeFallEnabled(true);

    // Attach the INT pin to an interrupt
    attachInterrupt(digitalPinToInterrupt(19), []() {
        Serial.println("Motion/FreeFall detected!");
    }, RISING);
}
```

---

## 2. HC-SR04 - Ultrasonic Sensor

### How it works

```
Measuring distance with sound waves:

ESP32 → TRIG pin: 10µs HIGH pulse
          ↓
     Sensor: sends 8 sound pulses at 40kHz
          ↓
     the waves hit an object and bounce back
          ↓
     ECHO pin: HIGH until the echo is received

Distance = (ECHO time / 2) × speed of sound
Speed of sound ≈ 343 m/s = 0.0343 cm/µs

Distance (cm) = time_µs × 0.0343 / 2
```

### Wiring

```
HC-SR04        ESP32
  VCC  ──────── 5V
  GND  ──────── GND
  TRIG ──────── GPIO5  (output)
  ECHO ──────── GPIO18 (input)

Problem: the ECHO pin is 5V → needs a voltage divider!
  ECHO ─── 10kΩ ─── GPIO18
                └── 20kΩ ─── GND
```

### Code

```cpp
#define TRIG_PIN 5
#define ECHO_PIN 18

void setup() {
    Serial.begin(115200);
    pinMode(TRIG_PIN, OUTPUT);
    pinMode(ECHO_PIN, INPUT);
    digitalWrite(TRIG_PIN, LOW);
}

float measureDistance() {
    // Trigger pulse
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);

    // Measure the ECHO duration
    long duration = pulseIn(ECHO_PIN, HIGH, 30000);  // timeout: 30ms
    // 30ms → max distance ~515cm

    if (duration == 0) {
        return -1.0;  // timeout - out of range
    }

    // Calculate distance
    float distance_cm = duration * 0.0343 / 2.0;
    return distance_cm;
}

void loop() {
    float dist = measureDistance();

    if (dist < 0) {
        Serial.println("Out of range!");
    } else {
        Serial.printf("Distance: %.1f cm\n", dist);
    }

    delay(100);
}
```

### Averaging for better accuracy

```cpp
float measureDistanceAvg(int samples = 5) {
    float sum = 0;
    int valid = 0;

    for (int i = 0; i < samples; i++) {
        float d = measureDistance();
        if (d > 0) {
            sum += d;
            valid++;
        }
        delay(20);
    }

    return valid > 0 ? sum / valid : -1.0;
}
```

---

## 3. PIR - Motion Sensor

### How the PIR works

```
PIR (Passive Infrared) detects a change in infrared radiation:

Warm objects (like a person) → emit IR
The sensor → compares IR across two zones (Fresnel lens)
A change → HIGH output

Specs:
  Range: 3-7 meters
  Angle: 100-120 degrees
  Output delay: 3-5 seconds (adjustable)
  Two trimmers: sensitivity and time delay
```

### Wiring and code

```cpp
#define PIR_PIN 27

// Callback for the interrupt
volatile bool motionDetected = false;

void IRAM_ATTR onMotion() {
    motionDetected = true;
}

void setup() {
    Serial.begin(115200);
    pinMode(PIR_PIN, INPUT);

    // Some PIRs need a warmup period
    Serial.println("PIR warming up (30 seconds)...");
    delay(30000);
    Serial.println("PIR ready");

    attachInterrupt(digitalPinToInterrupt(PIR_PIN),
                    onMotion, RISING);
}

void loop() {
    if (motionDetected) {
        motionDetected = false;
        Serial.printf("[%lu] Motion detected!\n", millis() / 1000);

        // Desired action
        digitalWrite(2, HIGH);  // turn the LED on
        delay(5000);
        digitalWrite(2, LOW);
    }
}
```

---

## 4. GPS Neo-6M - Geographic Position

### Neo-6M Specs

```
Protocol: UART (9600 baud by default)
Output: NMEA 0183 sentences
Position accuracy: 2.5 meters CEP
Time accuracy: 30 nanoseconds (!)
Cold start time: 27 seconds
Hot start time: 1 second
Voltage: 3.3V or 5V
```

### Wiring

```
Neo-6M         ESP32
  VCC  ──────── 3.3V or 5V
  GND  ──────── GND
  TX   ──────── GPIO16 (RX2)
  RX   ──────── GPIO17 (TX2)
```

### The NMEA protocol

```
NMEA Sentences:
$GPGGA: position, altitude, quality
$GPRMC: position + speed + date
$GPGSV: satellites in view
$GPVTG: speed and heading
$GPGSA: DOP and active satellites

Example $GPRMC:
$GPRMC,123519,A,4807.038,N,01131.000,E,022.4,084.4,230394,003.1,W*6A

  123519  → time 12:35:19 UTC
  A       → Valid
  4807.038,N → Latitude: 48°07.038'N
  01131.000,E → Longitude: 011°31.000'E
  022.4   → speed: 22.4 knots
  084.4   → heading: 84.4 degrees
  230394  → date: 23/03/1994
```

### Code with TinyGPS++

```cpp
#include <HardwareSerial.h>
#include <TinyGPSPlus.h>

TinyGPSPlus gps;
HardwareSerial gpsSerial(2);  // UART2

void setup() {
    Serial.begin(115200);
    gpsSerial.begin(9600, SERIAL_8N1, 16, 17);  // RX=16, TX=17
    Serial.println("GPS initialized. Waiting for fix...");
}

void loop() {
    // Read and parse GPS data
    while (gpsSerial.available()) {
        gps.encode(gpsSerial.read());
    }

    if (gps.location.isUpdated()) {
        Serial.printf("Location: %.6f, %.6f\n",
                       gps.location.lat(),
                       gps.location.lng());
        Serial.printf("Altitude: %.1f m\n", gps.altitude.meters());
        Serial.printf("Speed: %.1f km/h\n", gps.speed.kmph());
        Serial.printf("Course: %.1f°\n", gps.course.deg());
        Serial.printf("Satellites: %d\n", gps.satellites.value());
        Serial.printf("HDOP: %.1f\n", gps.hdop.value() / 100.0);

        if (gps.date.isValid() && gps.time.isValid()) {
            Serial.printf("UTC: %04d-%02d-%02d %02d:%02d:%02d\n",
                           gps.date.year(), gps.date.month(),
                           gps.date.day(), gps.time.hour(),
                           gps.time.minute(), gps.time.second());
        }
    }

    if (gps.satellites.value() == 0 && millis() > 10000) {
        Serial.println("No GPS fix yet. Move to open sky.");
    }

    delay(1000);
}
```

### Calculating the distance between two GPS points

```cpp
// Haversine formula
double calculateDistance(double lat1, double lon1,
                          double lat2, double lon2) {
    const double R = 6371000;  // Earth's radius (meters)
    double phi1   = lat1 * PI / 180;
    double phi2   = lat2 * PI / 180;
    double dphi   = (lat2 - lat1) * PI / 180;
    double dlambda = (lon2 - lon1) * PI / 180;

    double a = sin(dphi/2) * sin(dphi/2) +
               cos(phi1) * cos(phi2) *
               sin(dlambda/2) * sin(dlambda/2);

    double c = 2 * atan2(sqrt(a), sqrt(1-a));
    return R * c;  // meters
}

// Example:
double dist = calculateDistance(35.6892, 51.3890,  // Tehran
                                 32.6546, 51.6680); // Isfahan
Serial.printf("Distance: %.0f km\n", dist / 1000);
```

---

---

# Chapter 5.3 - Visual & Optical Sensors

---

## 1. ESP32-CAM - OV2640 Camera

### Specs

```
ESP32-CAM board:
  MCU: ESP32-S Module
  Camera: OV2640
  Flash: 4MB
  PSRAM: 4MB (for the frame buffer)
  MicroSD slot
  Flash LED (GPIO4)

OV2640 specs:
  Resolution: up to 2MP (UXGA: 1600×1200)
  Format: JPEG, BMP, YUV, RGB
  Protocol: SCCB (similar to I2C)
  Frame rate: 30fps at SVGA, 15fps at UXGA

Selectable resolutions:
  FRAMESIZE_96X96   → 96×96
  FRAMESIZE_QQVGA   → 160×120
  FRAMESIZE_QVGA    → 320×240
  FRAMESIZE_VGA     → 640×480
  FRAMESIZE_SVGA    → 800×600
  FRAMESIZE_XGA     → 1024×768
  FRAMESIZE_UXGA    → 1600×1200
```

### Code - MJPEG Stream

```cpp
#include "esp_camera.h"
#include <WiFi.h>
#include "esp_http_server.h"

// ESP32-CAM pins (AI-Thinker)
#define PWDN_GPIO_NUM  32
#define RESET_GPIO_NUM -1
#define XCLK_GPIO_NUM   0
#define SIOD_GPIO_NUM  26
#define SIOC_GPIO_NUM  27
#define Y9_GPIO_NUM    35
#define Y8_GPIO_NUM    34
#define Y7_GPIO_NUM    39
#define Y6_GPIO_NUM    36
#define Y5_GPIO_NUM    21
#define Y4_GPIO_NUM    19
#define Y3_GPIO_NUM    18
#define Y2_GPIO_NUM     5
#define VSYNC_GPIO_NUM 25
#define HREF_GPIO_NUM  23
#define PCLK_GPIO_NUM  22

camera_config_t camera_config = {
    .pin_pwdn  = PWDN_GPIO_NUM,
    .pin_reset = RESET_GPIO_NUM,
    .pin_xclk  = XCLK_GPIO_NUM,
    .pin_sccb_sda = SIOD_GPIO_NUM,
    .pin_sccb_scl = SIOC_GPIO_NUM,
    .pin_d7 = Y9_GPIO_NUM,
    .pin_d6 = Y8_GPIO_NUM,
    .pin_d5 = Y7_GPIO_NUM,
    .pin_d4 = Y6_GPIO_NUM,
    .pin_d3 = Y5_GPIO_NUM,
    .pin_d2 = Y4_GPIO_NUM,
    .pin_d1 = Y3_GPIO_NUM,
    .pin_d0 = Y2_GPIO_NUM,
    .pin_vsync = VSYNC_GPIO_NUM,
    .pin_href  = HREF_GPIO_NUM,
    .pin_pclk  = PCLK_GPIO_NUM,
    .xclk_freq_hz = 20000000,
    .ledc_timer   = LEDC_TIMER_0,
    .ledc_channel = LEDC_CHANNEL_0,
    .pixel_format = PIXFORMAT_JPEG,
    .frame_size   = FRAMESIZE_VGA,
    .jpeg_quality = 12,  // 0-63, lower = higher quality
    .fb_count     = 2,   // needs PSRAM
    .grab_mode    = CAMERA_GRAB_WHEN_EMPTY
};

void setup() {
    Serial.begin(115200);

    // Initialize the camera
    esp_err_t err = esp_camera_init(&camera_config);
    if (err != ESP_OK) {
        Serial.printf("Camera init failed: 0x%x\n", err);
        return;
    }
    Serial.println("Camera initialized");

    // Image settings
    sensor_t *s = esp_camera_sensor_get();
    s->set_brightness(s, 0);      // -2 to 2
    s->set_contrast(s, 0);        // -2 to 2
    s->set_saturation(s, 0);      // -2 to 2
    s->set_special_effect(s, 0);  // 0=None, 1=Negative, 2=Grayscale
    s->set_whitebal(s, 1);        // white balance
    s->set_hmirror(s, 0);         // mirror
    s->set_vflip(s, 0);           // flip

    WiFi.begin("SSID", "Password");
    while (WiFi.status() != WL_CONNECTED) delay(500);
    Serial.printf("Camera stream: http://%s/stream\n",
                   WiFi.localIP().toString().c_str());
}

// MJPEG stream handler
esp_err_t stream_handler(httpd_req_t *req) {
    camera_fb_t *fb = NULL;
    esp_err_t res = ESP_OK;

    httpd_resp_set_type(req, "multipart/x-mixed-replace; boundary=frame");

    while (true) {
        fb = esp_camera_fb_get();
        if (!fb) { res = ESP_FAIL; break; }

        char part_buf[64];
        size_t hlen = snprintf(part_buf, sizeof(part_buf),
            "--frame\r\nContent-Type: image/jpeg\r\n"
            "Content-Length: %zu\r\n\r\n", fb->len);

        res = httpd_resp_send_chunk(req, part_buf, hlen);
        res = httpd_resp_send_chunk(req, (char*)fb->buf, fb->len);
        res = httpd_resp_send_chunk(req, "\r\n", 2);

        esp_camera_fb_return(fb);

        if (res != ESP_OK) break;
    }
    return res;
}
```

### Capturing a photo and saving it to an SD card

```cpp
#include "FS.h"
#include "SD_MMC.h"

void captureAndSave() {
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) { Serial.println("Camera capture failed"); return; }

    // Filename with a timestamp
    char filename[32];
    snprintf(filename, sizeof(filename),
             "/photo_%lu.jpg", millis());

    // Save to SD
    File file = SD_MMC.open(filename, FILE_WRITE);
    if (!file) {
        Serial.println("SD write failed");
    } else {
        file.write(fb->buf, fb->len);
        file.close();
        Serial.printf("Saved: %s (%d bytes)\n",
                       filename, fb->len);
    }

    esp_camera_fb_return(fb);
}
```

---

## 2. IR Sensors - Infrared

### Types of IR sensors

```
1. IR Obstacle Sensor:
   threshold-based, for detecting obstacles
   output: digital HIGH/LOW

2. IR Distance Sensor (Sharp GP2Y):
   analog, measures distance
   range: 10-80cm (depending on model)

3. IR Receiver (TSOP38238):
   receives remote-control signals
   frequency: 38kHz
   protocol: NEC, Sony, RC5, ...
```

### IR Remote Control

```cpp
#include <IRremote.hpp>

#define IR_RECEIVE_PIN 15

void setup() {
    Serial.begin(115200);
    IrReceiver.begin(IR_RECEIVE_PIN, ENABLE_LED_FEEDBACK);
    Serial.println("IR receiver ready");
}

void loop() {
    if (IrReceiver.decode()) {
        // Show the received info
        Serial.printf("Protocol: %s\n",
                       IrReceiver.decodedIRData.protocol == NEC ? "NEC" :
                       IrReceiver.decodedIRData.protocol == SONY ? "SONY" :
                       "Unknown");
        Serial.printf("Address: 0x%04X\n",
                       IrReceiver.decodedIRData.address);
        Serial.printf("Command: 0x%02X\n",
                       IrReceiver.decodedIRData.command);

        // Handle the command
        switch (IrReceiver.decodedIRData.command) {
            case 0x45: Serial.println("POWER"); break;
            case 0x46: Serial.println("VOL+"); break;
            case 0x15: Serial.println("VOL-"); break;
        }

        IrReceiver.resume();
    }
}
```

### IR Transmit (sending)

```cpp
#include <IRremote.hpp>

#define IR_SEND_PIN 4

void setup() {
    IrSender.begin(IR_SEND_PIN);
}

void sendIR() {
    // Send the NEC protocol
    IrSender.sendNEC(0x00FF, 0x12, 3);  // address, command, repeats

    // Send Samsung
    IrSender.sendSamsung(0x07, 0x02, 1);

    // Send a raw signal
    uint16_t rawData[] = {9000, 4500, 560, 560, 560, 1690, ...};
    IrSender.sendRaw(rawData,
                     sizeof(rawData) / sizeof(rawData[0]),
                     38);  // 38kHz
}
```

---

## 3. LDR - Light Sensor

```cpp
// LDR (Light Dependent Resistor)
// resistance in darkness: ~1MΩ
// resistance in light: ~100Ω

#define LDR_PIN 34

void setup() {
    Serial.begin(115200);
    analogReadResolution(12);
    analogSetAttenuation(ADC_11db);
}

void loop() {
    int raw = analogRead(LDR_PIN);
    float voltage = raw * 3.3 / 4095.0;

    // Calculate LDR resistance (voltage divider with R=10kΩ)
    float ldr_resistance = (3.3 - voltage) / voltage * 10000;  // Ω

    // Convert to Lux (approximate)
    float lux = 500.0 / (ldr_resistance / 1000.0);

    Serial.printf("Raw: %d | V: %.2fV | R: %.0fΩ | Lux: %.1f\n",
                   raw, voltage, ldr_resistance, lux);

    // Day/night detection
    if (raw > 3000)       Serial.println("Very bright");
    else if (raw > 2000)  Serial.println("Bright");
    else if (raw > 1000)  Serial.println("Dim");
    else                  Serial.println("Dark");

    delay(500);
}
```

---

---

# Chapter 5.4 - Communicating with External Devices

---

## 1. RFID RC522

### What is RFID?

```
RFID (Radio Frequency Identification):
  Frequency: 13.56 MHz (RC522)
  Range: ~5 centimeters
  Protocol: ISO/IEC 14443A

Tag types:
  MIFARE Classic 1K:  1KB memory, 16 sectors, 4-byte UID
  MIFARE Classic 4K:  4KB memory
  MIFARE Ultralight:  64 bytes, low cost
  NTAG213/215/216:    NFC/RFID, a few hundred bytes

MIFARE Classic 1K structure:
  16 sectors × 4 blocks × 16 bytes = 1024 bytes
  Block 0 (Sector 0): UID and manufacturer data (read-only)
  Last block of each sector: Sector Trailer (Key A + Access Bits + Key B)
```

### RC522 Wiring

```
RC522          ESP32
  SDA(SS) ──── GPIO5  (CS)
  SCK     ──── GPIO18 (SCLK)
  MOSI    ──── GPIO23 (MOSI)
  MISO    ──── GPIO19 (MISO)
  IRQ     ──── (optional)
  GND     ──── GND
  RST     ──── GPIO22
  3.3V    ──── 3.3V (important: not 5V!)
```

### Code

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN  5
#define RST_PIN 22

MFRC522 rfid(SS_PIN, RST_PIN);
MFRC522::MIFARE_Key key;

void setup() {
    Serial.begin(115200);
    SPI.begin(18, 19, 23, 5);  // SCK, MISO, MOSI, SS
    rfid.PCD_Init();

    // MIFARE default key
    for (byte i = 0; i < 6; i++) key.keyByte[i] = 0xFF;

    Serial.println("RFID reader ready. Scan card...");
}

void loop() {
    // Check for a new card
    if (!rfid.PICC_IsNewCardPresent()) return;
    if (!rfid.PICC_ReadCardSerial()) return;

    // Read the UID
    Serial.print("UID: ");
    String uid = "";
    for (byte i = 0; i < rfid.uid.size; i++) {
        Serial.printf("%02X ", rfid.uid.uidByte[i]);
        uid += String(rfid.uid.uidByte[i], HEX);
    }
    Serial.println();

    // Card type
    MFRC522::PICC_Type piccType = rfid.PICC_GetType(rfid.uid.sak);
    Serial.printf("Card type: %s\n",
                   rfid.PICC_GetTypeName(piccType));

    // Read block 1 (Sector 0, Block 1)
    byte block = 1;
    byte buffer[18];
    byte size = sizeof(buffer);

    // Authenticate with Key A
    MFRC522::StatusCode status = rfid.PCD_Authenticate(
        MFRC522::PICC_CMD_MF_AUTH_KEY_A, block, &key, &(rfid.uid));

    if (status == MFRC522::STATUS_OK) {
        status = rfid.MIFARE_Read(block, buffer, &size);
        if (status == MFRC522::STATUS_OK) {
            Serial.print("Block 1: ");
            for (int i = 0; i < 16; i++) {
                Serial.printf("%02X ", buffer[i]);
            }
            Serial.println();
        }
    } else {
        Serial.printf("Auth failed: %s\n",
                       rfid.GetStatusCodeName(status));
    }

    // Write to block 2
    byte dataToWrite[16] = {
        'H','e','l','l','o',' ','E','S','P','3','2','!',
        0,0,0,0
    };

    status = rfid.PCD_Authenticate(
        MFRC522::PICC_CMD_MF_AUTH_KEY_A, 2, &key, &(rfid.uid));

    if (status == MFRC522::STATUS_OK) {
        status = rfid.MIFARE_Write(2, dataToWrite, 16);
        Serial.printf("Write: %s\n",
            status == MFRC522::STATUS_OK ? "OK" : "Failed");
    }

    rfid.PICC_HaltA();
    rfid.PCD_StopCrypto1();
}
```

---

## 2. PN532 - NFC Reader

### RC522 vs. PN532

```
RC522:
  RFID 13.56MHz read/write only
  Interface: SPI

PN532:
  full RFID + NFC
  NDEF read/write
  Card emulation
  Peer-to-Peer
  Interface: I2C, SPI, or UART (selected via a DIP switch)
```

### PN532 code with I2C

```cpp
#include <Wire.h>
#include <Adafruit_PN532.h>

#define PN532_IRQ   2
#define PN532_RESET 3

Adafruit_PN532 nfc(PN532_IRQ, PN532_RESET);

void setup() {
    Serial.begin(115200);
    nfc.begin();

    uint32_t versiondata = nfc.getFirmwareVersion();
    if (!versiondata) {
        Serial.println("PN532 not found!");
        while (1);
    }

    Serial.printf("PN532 Firmware: v%d.%d\n",
                   (versiondata >> 16) & 0xFF,
                   (versiondata >> 8) & 0xFF);

    nfc.SAMConfig();
    Serial.println("Waiting for NFC card...");
}

void loop() {
    uint8_t uid[7];
    uint8_t uidLen;

    bool found = nfc.readPassiveTargetID(
        PN532_MIFARE_ISO14443A, uid, &uidLen, 500);  // 500ms timeout

    if (found) {
        Serial.print("UID: ");
        for (int i = 0; i < uidLen; i++)
            Serial.printf("%02X ", uid[i]);
        Serial.println();

        // Read NDEF from an NTAG
        if (uidLen == 7) {  // NTAG
            uint8_t data[32];
            bool ok = nfc.ntag2xx_ReadPage(4, data);  // page 4 is where NDEF starts
            if (ok) {
                Serial.print("NDEF data: ");
                for (int i = 0; i < 4; i++)
                    Serial.printf("%02X ", data[i]);
                Serial.println();
            }
        }
    }
}
```

---

## 3. Fingerprint Sensor

### Specs

```
Common modules:
  R307, R503 (Optical)
  AS608
  FPM10A

Protocol: UART
Connections: VCC, GND, TX, RX
Capacity: 150-1000 fingerprints (depending on model)
Security: the template never leaves the sensor
```

### Code - Adafruit Fingerprint library

```cpp
#include <Adafruit_Fingerprint.h>
#include <HardwareSerial.h>

HardwareSerial mySerial(2);  // UART2
Adafruit_Fingerprint finger(&mySerial);

void setup() {
    Serial.begin(115200);
    mySerial.begin(57600, SERIAL_8N1, 16, 17);  // RX=16, TX=17

    if (finger.begin(0xFFFFFFFF)) {  // default password
        Serial.println("Fingerprint sensor ready");
        Serial.printf("Capacity: %d\n", finger.capacity);
        Serial.printf("Security level: %d\n", finger.security_level);
    } else {
        Serial.println("Fingerprint sensor not found!");
    }
}

// Enroll a new fingerprint
bool enrollFingerprint(int id) {
    Serial.printf("Enrolling ID #%d\n", id);

    // First step: scan
    Serial.println("Place finger...");
    while (finger.getImage() != FINGERPRINT_OK) delay(100);

    if (finger.image2Tz(1) != FINGERPRINT_OK) {
        Serial.println("Conversion failed");
        return false;
    }

    // Second step: scan again
    Serial.println("Remove finger...");
    delay(2000);
    Serial.println("Place same finger again...");
    while (finger.getImage() != FINGERPRINT_OK) delay(100);

    if (finger.image2Tz(2) != FINGERPRINT_OK) {
        Serial.println("Conversion failed");
        return false;
    }

    // Combine and store
    if (finger.createModel() != FINGERPRINT_OK) {
        Serial.println("Model creation failed - fingers don't match");
        return false;
    }

    if (finger.storeModel(id) != FINGERPRINT_OK) {
        Serial.println("Store failed");
        return false;
    }

    Serial.printf("Fingerprint #%d enrolled!\n", id);
    return true;
}

// Verify a fingerprint
int verifyFingerprint() {
    if (finger.getImage() != FINGERPRINT_OK) return -1;
    if (finger.image2Tz() != FINGERPRINT_OK) return -1;
    if (finger.fingerSearch() != FINGERPRINT_OK) {
        Serial.println("Not found");
        return -1;
    }

    Serial.printf("Match! ID #%d, confidence: %d\n",
                   finger.fingerID, finger.confidence);
    return finger.fingerID;
}

void loop() {
    int id = verifyFingerprint();
    if (id >= 0) {
        // Access granted
        Serial.printf("Access granted for user %d\n", id);
        digitalWrite(2, HIGH);
        delay(3000);
        digitalWrite(2, LOW);
    }
    delay(100);
}
```

---

## 4. Keypad Matrix

### How a matrix keypad works

```
4×4 Keypad:
  4 rows + 4 columns = 8 pins for 16 keys

Matrix:
       C1   C2   C3   C4
  R1 [  1 ][ 2 ][ 3 ][ A ]
  R2 [  4 ][ 5 ][ 6 ][ B ]
  R3 [  7 ][ 8 ][ 9 ][ C ]
  R4 [  * ][ 0 ][ # ][ D ]

Detecting a keypress:
  drive each row LOW one at a time
  whichever column reads LOW = the pressed key
```

### Code

```cpp
#include <Keypad.h>

const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] = {
    {'1','2','3','A'},
    {'4','5','6','B'},
    {'7','8','9','C'},
    {'*','0','#','D'}
};

byte rowPins[ROWS] = {13, 12, 14, 27};
byte colPins[COLS] = {26, 25, 33, 32};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

String password = "1234";
String input    = "";

void setup() {
    Serial.begin(115200);
    Serial.println("Enter password:");
}

void loop() {
    char key = keypad.getKey();

    if (key) {
        Serial.print(key);

        if (key == '#') {
            // Confirm
            if (input == password) {
                Serial.println("\nAccess Granted!");
                digitalWrite(2, HIGH);
                delay(3000);
                digitalWrite(2, LOW);
            } else {
                Serial.println("\nWrong password!");
            }
            input = "";
        } else if (key == '*') {
            // Clear
            input = "";
            Serial.println("\nCleared");
        } else {
            input += key;
        }
    }
}
```

---

## 5. LCD/OLED - SSD1306

### SSD1306 OLED

```
Specs:
  Size: 0.96" or 1.3"
  Resolution: 128×64 pixels
  Color: monochrome (white, blue, or yellow+blue)
  Protocol: I2C (address 0x3C or 0x3D) or SPI
  Consumption: very low
```

### Code - Adafruit SSD1306 library

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1   // Reset pin (or -1 if sharing Arduino reset pin)

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT,
                          &Wire, OLED_RESET);

void setup() {
    Serial.begin(115200);
    Wire.begin(21, 22);

    if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
        Serial.println("SSD1306 not found!");
        while (1);
    }

    // Clear
    display.clearDisplay();

    // Text
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 0);
    display.println("ESP32 IoT Node");
    display.println("---------------");

    // Sensor data
    display.setTextSize(2);
    display.setCursor(0, 20);
    display.printf("T: 25.5 C");
    display.setCursor(0, 44);
    display.printf("H: 60 %%");

    display.display();  // send to the OLED
}

void updateDisplay(float temp, float hum, bool wifi) {
    display.clearDisplay();

    // Title
    display.setTextSize(1);
    display.setCursor(0, 0);
    display.print("IoT Monitor");
    display.setCursor(100, 0);
    display.print(wifi ? "W" : "X");  // WiFi indicator

    // Divider line
    display.drawLine(0, 9, 127, 9, SSD1306_WHITE);

    // Data
    display.setTextSize(2);
    display.setCursor(0, 14);
    display.printf("%.1fC", temp);
    display.setCursor(0, 38);
    display.printf("%.0f%%", hum);

    // Progress bar for humidity
    int barWidth = (int)(hum / 100.0 * 128);
    display.drawRect(0, 57, 128, 7, SSD1306_WHITE);
    display.fillRect(0, 57, barWidth, 7, SSD1306_WHITE);

    display.display();
}
```

---

---

# Chapter 5.5 - Actuators

---

## Introduction: What is an Actuator?

An actuator is the opposite of a sensor — it converts an electrical signal into **physical motion or action**:

```
ESP32 → digital/PWM signal → [Actuator] → motion/sound/light

Types:
  Relay:        switching AC/DC current on and off
  Servo:        precise angular rotation (0-180°)
  Stepper:      precise step-by-step rotation
  DC motor:     continuous rotation at variable speed
  LED/Buzzer:   light/sound
```

---

## 1. Relay and AC Control

### What is a relay?

```
A relay is an electromagnetic switch:
  Control: a 3.3V/5V signal from the ESP32
  Output: switches an AC circuit (220V!) or high-current DC on/off

Types:
  Normally Open (NO): open (disconnected) by default
  Normally Closed (NC): closed (connected) by default
  Common (COM): the shared pin

A relay module usually has:
  IN pin: control from the ESP32
  VCC, GND: power
  NO, COM, NC: output

Important safety note:
  ⚠️ Never work directly with 220V without proper training!
  Use isolated modules
```

### Wiring

```
Relay module     ESP32
  VCC   ──────── 5V (some are 3.3V)
  GND   ──────── GND
  IN    ──────── GPIO26

Relay output (for a 220V lamp):
  COM   ──── one lamp terminal
  NO    ──── 220V live (when IN=LOW, lamp on)

or:
  NC    ──── 220V live (when IN=LOW, lamp off)
```

### Code

```cpp
#define RELAY_PIN 26

// Most relay modules are Active Low:
// LOW → relay ON
// HIGH → relay OFF
#define RELAY_ON  LOW
#define RELAY_OFF HIGH

void setup() {
    Serial.begin(115200);
    pinMode(RELAY_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, RELAY_OFF);  // start off
    Serial.println("Relay initialized (OFF)");
}

// Control via Serial
void loop() {
    if (Serial.available()) {
        String cmd = Serial.readStringUntil('\n');
        cmd.trim();

        if (cmd == "on") {
            digitalWrite(RELAY_PIN, RELAY_ON);
            Serial.println("Relay ON");
        } else if (cmd == "off") {
            digitalWrite(RELAY_PIN, RELAY_OFF);
            Serial.println("Relay OFF");
        } else if (cmd.startsWith("timer ")) {
            int secs = cmd.substring(6).toInt();
            Serial.printf("Timer %d seconds\n", secs);
            digitalWrite(RELAY_PIN, RELAY_ON);
            delay(secs * 1000);
            digitalWrite(RELAY_PIN, RELAY_OFF);
            Serial.println("Timer done");
        }
    }
}
```

### Multi-channel relay

```cpp
// 4-channel relay module
#define RELAY1 26
#define RELAY2 27
#define RELAY3 14
#define RELAY4 12

int relayPins[] = {RELAY1, RELAY2, RELAY3, RELAY4};
int numRelays = 4;

void setup() {
    for (int i = 0; i < numRelays; i++) {
        pinMode(relayPins[i], OUTPUT);
        digitalWrite(relayPins[i], HIGH);  // all off
    }
}

void setRelay(int channel, bool state) {
    if (channel < 1 || channel > numRelays) return;
    digitalWrite(relayPins[channel - 1],
                  state ? LOW : HIGH);  // Active Low
}
```

---

## 2. Servo Motor

### What is a servo?

```
Servo motor = DC motor + gearbox + potentiometer + controller

Angle: 0-180 degrees (Standard) or continuous rotation (Continuous)
Control: PWM with a 20ms period (50Hz)
  Signal: 1ms → 0°
  Signal: 1.5ms → 90°
  Signal: 2ms → 180°

Power: 5V (separate from the ESP32 for larger servos)
```

### Code

```cpp
#include <ESP32Servo.h>

Servo myServo;

#define SERVO_PIN 18

void setup() {
    Serial.begin(115200);

    // Allocate a timer for the servo
    ESP32PWM::allocateTimer(0);

    myServo.setPeriodHertz(50);    // Standard 50Hz servo
    myServo.attach(SERVO_PIN, 1000, 2000);
    // 1000µs = 0° and 2000µs = 180°
}

void loop() {
    // Slow sweep back and forth
    for (int pos = 0; pos <= 180; pos += 1) {
        myServo.write(pos);
        delay(15);
    }
    delay(500);

    for (int pos = 180; pos >= 0; pos -= 1) {
        myServo.write(pos);
        delay(15);
    }
    delay(500);
}
```

### Controlling several servos

```cpp
#include <ESP32Servo.h>

Servo servo1, servo2, servo3;

void setup() {
    ESP32PWM::allocateTimer(0);
    ESP32PWM::allocateTimer(1);
    ESP32PWM::allocateTimer(2);

    servo1.attach(18);
    servo2.attach(19);
    servo3.attach(21);

    // Initial position
    servo1.write(90);
    servo2.write(90);
    servo3.write(90);
}
```

---

## 3. Stepper Motor with A4988

### What is a stepper motor?

```
Stepper Motor:
  precise rotation in fixed "steps"
  usually 200 steps per revolution (1.8° per step)
  with microstepping: down to 1/16 step = 0.1125°

A4988 Driver:
  controls the stepper via STEP and DIR pins
  Microstepping: Full, 1/2, 1/4, 1/8, 1/16
  Current: up to 2A
  Motor voltage: 8-35V (separate from the ESP32)
```

### A4988 Wiring

```
A4988           ESP32            Stepper
  DIR    ──── GPIO26
  STEP   ──── GPIO27
  SLEEP  ──── GPIO14
  RESET  ──── GPIO12
  MS1    ──── GPIO13 (microstepping)
  MS2    ──── GPIO33
  MS3    ──── GPIO32
  GND    ──── GND
  VDD    ──── 3.3V
  VMOT   ──── 12V (separate)
  GND    ──── motor GND
  1A,1B  ──── motor Coil A
  2A,2B  ──── motor Coil B
```

### Code

```cpp
#define STEP_PIN  27
#define DIR_PIN   26
#define SLEEP_PIN 14

void setup() {
    Serial.begin(115200);
    pinMode(STEP_PIN, OUTPUT);
    pinMode(DIR_PIN, OUTPUT);
    pinMode(SLEEP_PIN, OUTPUT);

    digitalWrite(SLEEP_PIN, HIGH);  // enable the driver
    delay(10);
}

void stepMotor(int steps, bool clockwise, int delayUs = 800) {
    digitalWrite(DIR_PIN, clockwise ? HIGH : LOW);

    for (int i = 0; i < abs(steps); i++) {
        digitalWrite(STEP_PIN, HIGH);
        delayMicroseconds(delayUs);
        digitalWrite(STEP_PIN, LOW);
        delayMicroseconds(delayUs);
    }
}

void loop() {
    // One full turn clockwise (200 steps)
    stepMotor(200, true, 800);
    delay(1000);

    // Half turn counter-clockwise (100 steps)
    stepMotor(100, false, 500);  // faster
    delay(1000);
}
```

### The AccelStepper library (acceleration and deceleration)

```cpp
#include <AccelStepper.h>

// DRIVER mode: just STEP and DIR
AccelStepper stepper(AccelStepper::DRIVER, 27, 26);

void setup() {
    stepper.setMaxSpeed(1000);       // steps per second
    stepper.setAcceleration(500);    // steps per second²
    stepper.moveTo(1000);            // move to position 1000
}

void loop() {
    stepper.run();  // must be called on every loop

    if (stepper.distanceToGo() == 0) {
        stepper.moveTo(-stepper.currentPosition());  // return
    }
}
```

---

## 4. DC Motor with L298N

### What is the L298N?

```
L298N Dual H-Bridge:
  controls 2 DC motors or 1 stepper
  motor voltage: 5-35V
  current: up to 2A per channel

Important pins:
  ENA: Enable for motor A (PWM for speed)
  IN1, IN2: direction for motor A
  ENB: Enable for motor B
  IN3, IN4: direction for motor B

Motor A truth table:
  ENA=HIGH, IN1=HIGH, IN2=LOW  → forward
  ENA=HIGH, IN1=LOW,  IN2=HIGH → reverse
  ENA=HIGH, IN1=HIGH, IN2=HIGH → brake
  ENA=LOW                       → free / coasting
```

### Wiring

```
L298N          ESP32
  ENA   ──── GPIO14 (PWM)
  IN1   ──── GPIO27
  IN2   ──── GPIO26
  ENB   ──── GPIO33 (PWM)
  IN3   ──── GPIO25
  IN4   ──── GPIO32
  GND   ──── shared GND
  5V    ──── ESP32 5V (low current)
  12V   ──── motor power supply
```

### Code

```cpp
// Motor A pins
#define ENA 14
#define IN1 27
#define IN2 26

// Motor B pins
#define ENB 33
#define IN3 25
#define IN4 32

// PWM channels
#define PWM_CH_A 0
#define PWM_CH_B 1
#define PWM_FREQ 1000
#define PWM_RES  8  // 8-bit: 0-255

void setup() {
    // Configure PWM
    ledcSetup(PWM_CH_A, PWM_FREQ, PWM_RES);
    ledcSetup(PWM_CH_B, PWM_FREQ, PWM_RES);
    ledcAttachPin(ENA, PWM_CH_A);
    ledcAttachPin(ENB, PWM_CH_B);

    // Direction pins
    pinMode(IN1, OUTPUT); pinMode(IN2, OUTPUT);
    pinMode(IN3, OUTPUT); pinMode(IN4, OUTPUT);
}

// Control motor A
void motorA(int speed) {
    // speed: -255 to +255
    if (speed > 0) {
        digitalWrite(IN1, HIGH);
        digitalWrite(IN2, LOW);
    } else if (speed < 0) {
        digitalWrite(IN1, LOW);
        digitalWrite(IN2, HIGH);
        speed = -speed;
    } else {
        digitalWrite(IN1, LOW);
        digitalWrite(IN2, LOW);
    }
    ledcWrite(PWM_CH_A, constrain(speed, 0, 255));
}

// Control motor B
void motorB(int speed) {
    if (speed > 0) {
        digitalWrite(IN3, HIGH);
        digitalWrite(IN4, LOW);
    } else if (speed < 0) {
        digitalWrite(IN3, LOW);
        digitalWrite(IN4, HIGH);
        speed = -speed;
    } else {
        digitalWrite(IN3, LOW);
        digitalWrite(IN4, LOW);
    }
    ledcWrite(PWM_CH_B, constrain(speed, 0, 255));
}

void loop() {
    // Forward at 70% speed
    motorA(178); motorB(178);
    delay(2000);

    // Brake
    motorA(0); motorB(0);
    delay(500);

    // Reverse at 50% speed
    motorA(-128); motorB(-128);
    delay(2000);

    // Turn right
    motorA(200); motorB(-200);
    delay(500);
}
```

---

---

# 🎯 End-of-Phase-5 Practical Project

## Project: ESP32 Smart Environment Monitor

```cpp
/*
 * Smart Environment Monitor
 * ─────────────────────────
 * Sensors: DHT22 + BMP280 + MQ-2
 * Display: SSD1306 OLED
 * Output: Relay (fan)
 * Communication: Serial + MQTT
 */

#include <Arduino.h>
#include <Wire.h>
#include <DHT.h>
#include <Adafruit_BME280.h>
#include <Adafruit_SSD1306.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// ────── Pins ──────
#define DHT_PIN     4
#define MQ2_PIN     34
#define RELAY_PIN   26
#define BUZZER_PIN  25

// ────── Objects ──────
DHT            dht(DHT_PIN, DHT22);
Adafruit_BME280 bme;
Adafruit_SSD1306 display(128, 64, &Wire, -1);

// ────── State ──────
struct Sensors {
    float temp_dht, humidity_dht;
    float temp_bmp, pressure;
    int   gas_raw;
    bool  gas_alarm;
} data;

// ────── Read sensors ──────
void readSensors() {
    // DHT22
    data.temp_dht    = dht.readTemperature();
    data.humidity_dht = dht.readHumidity();

    // BMP280
    data.temp_bmp = bme.readTemperature();
    data.pressure = bme.readPressure() / 100.0;

    // MQ-2
    data.gas_raw   = analogRead(MQ2_PIN);
    data.gas_alarm = data.gas_raw > 2000;

    // Fan control
    if (data.gas_alarm || data.temp_dht > 30) {
        digitalWrite(RELAY_PIN, LOW);   // fan on
        if (data.gas_alarm)
            tone(BUZZER_PIN, 1000, 500);  // alarm
    } else {
        digitalWrite(RELAY_PIN, HIGH);  // fan off
    }
}

// ────── OLED display ──────
void updateDisplay() {
    display.clearDisplay();

    display.setTextSize(1);
    display.setCursor(0, 0);
    display.printf("T:%.1fC H:%.0f%%", data.temp_dht, data.humidity_dht);
    display.setCursor(0, 10);
    display.printf("P:%.0fhPa", data.pressure);

    // Show gas level as a bar
    display.setCursor(0, 22);
    display.print("GAS: ");
    int barW = map(data.gas_raw, 0, 4095, 0, 80);
    display.fillRect(32, 22, barW, 8,
        data.gas_alarm ? SSD1306_WHITE : SSD1306_WHITE);
    display.drawRect(32, 22, 80, 8, SSD1306_WHITE);

    // Status
    display.setCursor(0, 34);
    display.printf("Fan:%s Gas:%s",
        digitalRead(RELAY_PIN) == LOW ? "ON " : "OFF",
        data.gas_alarm ? "ALM" : "OK ");

    // Uptime
    display.setCursor(0, 56);
    unsigned long up = millis() / 1000;
    display.printf("%02lu:%02lu:%02lu",
        up/3600, (up%3600)/60, up%60);

    display.display();
}

// ────── JSON Payload ──────
String buildJSON() {
    DynamicJsonDocument doc(512);
    doc["temp_dht"]     = data.temp_dht;
    doc["humidity"]     = data.humidity_dht;
    doc["temp_bmp"]     = data.temp_bmp;
    doc["pressure"]     = data.pressure;
    doc["gas_raw"]      = data.gas_raw;
    doc["gas_alarm"]    = data.gas_alarm;
    doc["fan_state"]    = digitalRead(RELAY_PIN) == LOW;
    doc["uptime"]       = millis() / 1000;

    String out; serializeJson(doc, out);
    return out;
}

void setup() {
    Serial.begin(115200);
    pinMode(RELAY_PIN, OUTPUT);
    pinMode(BUZZER_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, HIGH);  // off

    Wire.begin(21, 22);
    dht.begin();
    bme.begin(0x76);
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);

    display.clearDisplay();
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 28);
    display.print("  Initializing...");
    display.display();

    WiFi.begin("SSID", "Password");
    while (WiFi.status() != WL_CONNECTED) delay(500);

    Serial.printf("IP: %s\n", WiFi.localIP().toString().c_str());
}

unsigned long lastRead    = 0;
unsigned long lastDisplay = 0;

void loop() {
    if (millis() - lastRead > 2000) {
        lastRead = millis();
        readSensors();

        Serial.println(buildJSON());
    }

    if (millis() - lastDisplay > 500) {
        lastDisplay = millis();
        updateDisplay();
    }
}
```

---

## Sensor quick-reference table

| Sensor | Protocol | Main pins | Voltage | Library |
|-------|---------|-------------|-------|----------|
| DHT22 | 1-Wire | DATA+10kΩ | 3.3/5V | DHT |
| BME280 | I2C/SPI | SDA,SCL | 3.3V | Adafruit BME280 |
| DS18B20 | 1-Wire | DATA+4.7kΩ | 3.3V | DallasTemperature |
| MQ-2 | Analog | AOUT→ADC | 5V | - |
| MPU6050 | I2C | SDA,SCL | 3.3V | MPU6050 |
| HC-SR04 | Digital | TRIG,ECHO | 5V | - |
| PIR | Digital | OUT | 5V | - |
| Neo-6M GPS | UART | TX,RX | 3.3/5V | TinyGPS++ |
| ESP32-CAM | Camera | 18 pins | 5V | esp_camera |
| RC522 RFID | SPI | SS,SCK,MOSI,MISO | 3.3V | MFRC522 |
| PN532 NFC | I2C/SPI | SDA,SCL | 3.3V | Adafruit PN532 |
| Fingerprint | UART | TX,RX | 5V | Adafruit Fingerprint |
| SSD1306 | I2C | SDA,SCL | 3.3V | Adafruit SSD1306 |
| Relay | Digital | IN | 5V | - |
| Servo | PWM | SIGNAL | 5V | ESP32Servo |
| Stepper A4988 | Digital | STEP,DIR | 3.3V | AccelStepper |
| DC L298N | PWM | ENA,IN1,IN2 | 5V | - |

---

# 📚 Phase 5 Additional Resources

## Documentation
- **Adafruit:** learn.adafruit.com ← the best sensor tutorials
- **RandomNerdTutorials:** randomnerdtutorials.com ← practical ESP32 tutorials
- **ESP32 Pinout:** ← docs.espressif.com

## Important datasheets
- DHT22: the official AM2302 datasheet
- BME280: Bosch Sensortec datasheet
- MPU6050: InvenSense datasheet
- MIFARE Classic: NXP AN10609

---

> **Closing note for Phase 5:**
> Sensors and actuators are the bridge between the ESP32 and the physical world. From a Red Team perspective, familiarity with these devices helps you understand what data a target IoT device **collects** and what it **controls** — both of which matter for threat modeling and attack planning.

---

*Notes written for a network security engineer and Red Teamer | Phase 5 of the ESP32 IoT Security path*
