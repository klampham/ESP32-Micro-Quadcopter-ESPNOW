# 🚁 ESP32-S3 Micro Quadcopter & ESP-NOW Transmitter

Drone with an **ESP32-S3-WROOM-1** board acting as both **Flight Controller + Receiver** using the **ESP-NOW Protocol** (<10ms latency) to communicate with a custom **ESP32 DevKit V1 Physical Transmitter**.

> **Course Project:** Introduction to Mechatronics (Nhập môn Cơ điện tử)  
> **University:** University of Engineering and Technology - VNU Hanoi (UET - VNU)  
> **Author:** Phạm Khánh Lâm — **Student ID:** `24021145` (Contribution: 100%)
<img width="1376" height="768" alt="image" src="https://github.com/user-attachments/assets/45cb0fbb-d394-4d40-b3e2-c1153bcf069a" />

---

## 📌 Project Overview (Tổng quan dự án)
Dự án thiết kế và chế tạo hoàn chỉnh hệ thống máy bay không người lái 4 cánh quạt cỡ nhỏ (**Micro Quadcopter - Cấu hình Quad-X**) và tay cầm điều khiển không dây:
* **Flight Controller + Receiver:** Sử dụng trực tiếp vi điều khiển `ESP32-S3-WROOM-1` chạy firmware tùy biến từ `esp-fc` ở tần số vòng lặp `~2000 Hz` (`Cycle Time: 502 µs`).
* **Wireless Protocol:** Giao tiếp trực tiếp ngang hàng qua giao thức **ESP-NOW (2.4 GHz)** với gói tin `12-byte struct` đạt độ trễ `< 10 ms`, tích hợp cơ chế ngắt an toàn **Failsafe** khi mất sóng.
* **Power & Noise Immunity:** Trang bị mạch tăng áp 5V kèm tụ bù `1000µF` chống sụt áp (Brown-out Reset) và mạch lọc khởi động trễ RC (`10kΩ + 100nF`) tại chân `RST` giúp đạt chỉ số `I2C Error = 0`.

---

## 🛠️ Hardware & 3D Frame (Phần cứng & Khung cơ khí)

### 1. Drone (Flight Controller Node)
* **MCU:** `ESP32-S3-WROOM-1` (Dual-Core LX7 @ 240MHz)
* **IMU Sensor:** `GY-521 MPU6050` (6-DOF Gyro + Accelerometer, I2C Fast-Mode 400kHz)
* **Actuators:** 4x `8520 Coreless Brushed Motors` (3.7V) + `55mm Propellers` (2x CW, 2x CCW)
* **Motor Driver Stage (Custom PCB 2x6cm):**
  * 4x `AO3400A` N-Channel MOSFETs ($I_D = 5.7\text{A}, R_{DS(on)} < 33\text{m}\Omega$)
  * 4x `B5819W` Schottky Flyback Diodes
  * 4x `10kΩ` Gate Pull-down Resistors (SMD 0805)
* **Power System:** `1S LiPo Battery` (3.7V, 800mAh, $\ge 25\text{C}$) + `5V Boost Converter` + `1000µF` Capacitor + RC Reset Filter (`10kΩ` + `100nF` Ceramic Capacitor).

### 2. Transmitter (TX Controller Node)
* **MCU:** `ESP32 DevKit V1` (ESP-WROOM-32)
* **Inputs:** 2x Dual-axis Analog Joystick Modules + 2x Tactile Push Buttons (`Arm/Disarm` & `Flight Mode`)
* **Power System:** 1x `18650 Li-ion Battery` + `5V Boost Converter Module`

### 3. 3D Printed Frame (Khung máy in 3D)
* File thiết kế khung cơ khí nằm trong thư mục phần cứng: **`Fors3_reinforced.stl`**.
* Khung máy được tinh chỉnh gia cố thêm các góc lượn chịu lực (structural fillets) ở chân 4 cánh tay đòn để chống gãy vỡ khi va chạm và tối ưu hóa vị trí đặt bo mạch giúp cân bằng trọng tâm (CG).

---

## 🔌 Connections / Pinout (Sơ đồ đấu nối)

### A. Drone Connections (`ESP32-S3-WROOM-1`)

![Drone Schematic](Schematic_Drone_ESP32S3.png)

| Component / Signal | ESP32-S3 Pin | Notes / Description |
| :--- | :---: | :--- |
| **Motor 1 (Rear Right - CCW)** | `GPIO 7` | High-frequency PWM (8kHz) -> AO3400A Gate (+ 10kΩ to GND) |
| **Motor 2 (Front Right - CW)** | `GPIO 35` | High-frequency PWM (8kHz) -> AO3400A Gate (+ 10kΩ to GND) |
| **Motor 3 (Rear Left - CW)** | `GPIO 12` | High-frequency PWM (8kHz) -> AO3400A Gate (+ 10kΩ to GND) |
| **Motor 4 (Front Left - CCW)** | `GPIO 11` | High-frequency PWM (8kHz) -> AO3400A Gate (+ 100nF/10kΩ) |
| **MPU6050 SCL** | `GPIO 10` | I2C Clock (400kHz Fast-Mode) |
| **MPU6050 SDA** | `GPIO 9` | I2C Data |
| **MPU6050 VCC & GND** | `3V3` & `GND` | Clean 3.3V regulated supply |
| **RC Reset Filter** | `RST` | `10kΩ` pull-up to `3V3` + `100nF` capacitor to `GND` |
| **5V Boost Converter Out** | `5V` & `GND` | Decoupled with `1000µF` electrolytic capacitor |

### B. Transmitter Connections (`ESP32 DevKit V1`)
> **Lưu ý kỹ thuật:** Toàn bộ chiết áp Joystick được đấu vào khối **ADC1** (`GPIO 32 - 35`) để không bị xung đột phần cứng khi bật sóng Wi-Fi / ESP-NOW.

| Control Channel | Joystick Axis | ESP32 DevKit V1 Pin | Signal Range |
| :--- | :--- | :---: | :--- |
| **Throttle (Ga)** | Left Joystick - Vertical (Y) | `GPIO 32` (ADC1) | `1000 - 2000 µs` |
| **Yaw (Xoay hướng)** | Left Joystick - Horizontal (X) | `GPIO 33` (ADC1) | `1000 - 2000 µs` (Mid: 1500) |
| **Pitch (Chúi / Ngửa)** | Right Joystick - Vertical (Y) | `GPIO 34` (ADC1) | `1000 - 2000 µs` (Mid: 1500) |
| **Roll (Nghiêng trái / phải)** | Right Joystick - Horizontal (X) | `GPIO 35` (ADC1) | `1000 - 2000 µs` (Mid: 1500) |
| **Arm / Disarm Button** | Push Button 1 | `GPIO 25` | `INPUT_PULLUP` |
| **Mode Button** | Push Button 2 | `GPIO 26` | `INPUT_PULLUP` (Angle / Acro) |

---

## 💻 Software & Setup (Phần mềm & Cài đặt)

### 1. Công cụ cần thiết (Requirements)
* **[VS Code](https://code.visualstudio.com/)** + Extension **[PlatformIO IDE](https://platformio.org/)** (Dùng để biên dịch và nạp firmware `esp-fc` cho mạch bay ESP32-S3, và tay điều khiển).
* **[Betaflight Configurator (v10.10.0)](https://github.com/betaflight/betaflight-configurator/releases)** (Giao diện trạm mặt đất để hiệu chuẩn gia tốc kế, kiểm tra chiều động cơ và tinh chỉnh PID).

### 2. Các bước nạp code và Cài đặt
1. **Nạp Firmware cho Drone:** Mở thư mục `Firmware Flight Controller` trong VS Code (PlatformIO), kết nối ESP32-S3 qua cổng USB và nhấn **Upload**.
2. **Nạp Firmware cho Tay cầm:** Mở code trong `For-Transmitter-ESP32`, cập nhật địa chỉ MAC của Drone (nếu dùng Unicast) và nạp vào board ESP32 DevKit V1.
3. **Cấu hình trên Betaflight Configurator:**
   * Cắm cáp USB từ ESP32-S3 vào máy tính, nhấn **Connect**.
   * Đặt drone trên mặt phẳng ngang, vào tab **Setup** nhấn **Calibrate Accelerometer**.
   * Vào tab **CLI**, dán cấu hình tinh chỉnh bên dưới và gõ `save`.

---

## ⚙️ Betaflight CLI Configuration (Thông số tinh chỉnh PID cho Motor 8520)

```text
# Disable Airmode to prevent ground bouncing on touchdown
feature -AIRMODE

# RC Deadband to eliminate ADC potentiometer jitter
set deadband = 6
set yaw_deadband = 6

# Cascade PID Tuning for 8520 Brushed Motors
set p_pitch = 38
set i_pitch = 50
set d_pitch = 14
set f_pitch = 0
set p_roll = 35
set i_roll = 45
set d_roll = 12
set f_roll = 0
set p_yaw = 40
set i_yaw = 45
set level_angle_strength = 75

# Throttle Curve for smooth indoor hovering
set thr_mid = 60
set thr_expo = 50

save
```
## 🙏 Credits & Acknowledgments (Nguồn tham khảo & Ghi công)
Dự án được nghiên cứu, phát triển và tối ưu hóa dựa trên các nền tảng mã nguồn mở và tài liệu kỹ thuật sau:
* **Flight Controller Firmware:** [esp-fc](https://github.com/rtlopez/esp-fc) by **rtlopez** — Nền tảng firmware điều khiển bay tối ưu hóa dành riêng cho vi điều khiển ESP32.
* **Ground Control Station:** [Betaflight Configurator](https://github.com/betaflight/betaflight-configurator) — Giao diện trạm mặt đất dùng để cấu hình tham số, hiệu chuẩn cảm biến và giám sát dữ liệu thời gian thực qua giao thức MSP.
* **Hardware & Architecture Inspiration:** [LegendofKARTIS/esp32-drone](https://github.com/LegendofKARTIS/esp32-drone) by **Kartis Legend** — Tham khảo sơ đồ khối mạch kích MOSFET và cấu trúc truyền thông không dây ESP-NOW giữa Tay cầm và Drone.
