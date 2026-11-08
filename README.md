# Thiết kế và Chế tạo Hệ thống Micro Quadcopter và Tay cầm Điều khiển Không dây sử dụng ESP32
**Design and Implementation of an ESP32-Based Micro Quadcopter and Wireless Physical Transmitter**

> **Học phần:** Nhập môn Cơ điện tử  
> **Trường:** Đại học Công nghệ - Đại học Quốc gia Hà Nội (UET - VNU)  
> **Sinh viên thực hiện:** Phạm Khánh Lâm - **MSSV:** 24021145 (Đóng góp: 100%)

---

## 1. Giới thiệu tổng quan (Overview)
Dự án nghiên cứu, thiết kế và chế tạo hoàn chỉnh một hệ thống máy bay không người lái 4 cánh quạt cỡ nhỏ (**Micro Quadcopter - Cấu hình Quad-X**) đi kèm **Tay cầm điều khiển vật lý** sử dụng giao thức truyền thông trực tiếp **ESP-NOW** ở băng tần 2.4GHz. 

Hệ thống giải quyết bài toán độ trễ tín hiệu của Wi-Fi truyền thống và hiện tượng bão hòa xung điều khiển trên khung máy siêu nhẹ dùng động cơ chổi than Coreless 8520:
- **Độ trễ truyền thông (TX -> RX):** `< 10 ms` (Tần số gửi gói tin 50Hz - 100Hz qua ESP-NOW).
- **Tần số vòng lặp điều khiển (Gyro/PID Loop):** `~2000 Hz` (`Cycle Time: 502 us` trên nhân Dual-Core ESP32-S3).
- **Chế độ cân bằng:** Bộ điều khiển Cascade PID (Angle Loop & Rate Loop) kết hợp đường cong ga phi tuyến (`Throttle Expo`) và cơ chế ngắt an toàn khi mất sóng (`Failsafe`).

---

## 2. Kiến trúc Phần cứng (Hardware Specifications)

### A. Phân hệ Máy bay (Flight Controller - RX Node)
- **Vi điều khiển trung tâm:** ESP32-S3-WROOM-1 (Dual-core Xtensa LX7 @ 240MHz).
- **Cảm biến quán tính (IMU):** Module GY-521 MPU6050 (Giao tiếp I2C Fast-Mode 400kHz, `I2C Error = 0`).
- **Cơ cấu chấp hành:** 04 Động cơ DC chổi than Coreless `8520` + Cánh quạt 2 lá `55mm` (2 CW, 2 CCW).
- **Tầng công suất PWM:** 04 MOSFET kênh N `AO3400A` + 04 Diode Schottky chống dòng ngược `B5819W` + 04 Điện trở xả Gate `10kΩ` (Tần số băm xung PWM `8kHz`).
- **Nguồn cấp & Chống nhiễu:** Pin LiPo 1S (`3.7V - 800mAh, >=25C`), mạch Boost 5V kèm tụ bù `1000uF` nuôi ESP32-S3 và mạch lọc RC khởi động trễ (`10kΩ + 100nF`) tại chân `RST`.
- **Khung cơ khí:** Khung in 3D thiết kế gia cố góc chịu lực (`Fors3_reinforced.stl`).

| Linh kiện trên Drone | Chân kết nối ESP32-S3 | Chức năng |
| :--- | :---: | :--- |
| **Motor 1** (Sau - Phải, CCW) | `GPIO 7` | Xuất xung PWM kênh 1 |
| **Motor 2** (Trước - Phải, CW) | `GPIO 35` | Xuất xung PWM kênh 2 |
| **Motor 3** (Sau - Trái, CW) | `GPIO 12` | Xuất xung PWM kênh 3 |
| **Motor 4** (Trước - Trái, CCW) | `GPIO 11` | Xuất xung PWM kênh 4 |
| **MPU6050 SCL / SDA** | `GPIO 10` / `GPIO 9` | Truyền nhận dữ liệu IMU qua I2C |
| **Mạch lọc RC Reset** | `RST` (Pull-up `3V3`, Cap `GND`) | Chống nhiễu EMI và đồng bộ khởi động IMU |

### B. Phân hệ Tay cầm điều khiển (Transmitter - TX Node)
- **Vi điều khiển:** Board ESP32 DevKit V1 (ESP-WROOM-32).
- **Đầu vào điều khiển:** 02 cụm Joystick Analog 2 trục (đấu nối vào khối **ADC1**: `GPIO 32, 33, 34, 35` để không xung đột với sóng ESP-NOW) + Nút nhấn vật lý `Arm/Disarm` và `Mode`.
- **Xử lý tín hiệu:** Bộ lọc trung bình trượt mũ (EMA) khử nhiễu chiết áp và tạo vùng chết (`Deadband`).
- **Nguồn cấp:** Cell pin Li-ion `18650` kết hợp module tăng áp DC-DC 5V.

---

## 3. Cấu trúc Kho lưu trữ (Repository Structure)
- `01_Flight_Controller_Firmware/`: Mã nguồn điều khiển bay dựa trên `esp-fc` (biên dịch qua PlatformIO).
- `02_Transmitter_Firmware/`: Mã nguồn đọc ADC Joystick và truyền gói tin ESP-NOW trên tay cầm.
- `03_Hardware_and_3D_CAD/`: Chứa file khung in 3D (`Fors3_reinforced.stl`), sơ đồ nguyên lý và bảng linh kiện.
- `04_Betaflight_Config/`: Chứa file thông số tinh chỉnh PID thực tế (`diff all`) và hình ảnh kiểm chứng trên Betaflight Configurator.
- `05_Documentation/`: File báo cáo toàn văn môn học.

---

## 4. Thông số Tinh chỉnh PID & Rate (CLI Dump cho Motor 8520)
```text
feature -AIRMODE
set deadband = 6
set yaw_deadband = 6
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
set thr_mid = 60
set thr_expo = 50
save
