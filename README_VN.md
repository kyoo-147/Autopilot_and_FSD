# Autopilot & Full Self-Driving (FSD) cho xe tự hành mô hình
*Dự án điều khiển xe tự hành mô hình với khả năng tự lái và giữ làn dựa trên ROS và Deep Learning*

---

*Đọc bản tiếng Anh của tài liệu này tại [README.md](README.md).*

---

<table>
  <tr>
    <td align="center"><b>Mô hình xe tự hành thực tế</b></td>
    <td align="center"><b>Không gian phát triển dự án</b></td>
  </tr>
  <tr>
    <td><img src="assets/model.jpeg" alt="Scale Model Car" width="450"/></td>
    <td><img src="assets/work.jpeg" alt="Development Work" width="450"/></td>
  </tr>
</table>

---

> [!NOTE]
> Dự án này đã được mã nguồn mở (open-source) và hiện không còn sử dụng cho mục đích thương mại. Đây là tài liệu học thuật hữu ích để nghiên cứu và phát triển xe tự hành sử dụng hệ điều hành Robot (ROS) và các mô hình học sâu (Deep Learning) trên phần cứng nhúng (Raspberry Pi & Arduino/Nucleo).

---

## 🏆 Lịch sử dự án & Thành tựu

Dự án này là mã nguồn mở cho hệ thống tự hành của đội tuyển tham dự cuộc thi **Bosch Future Mobility Challenge (BFMC) 2024 & 2025**, hiện đang xếp hạng **Top 15 thế giới**.

Vào năm 2024, do một số trục trặc ngoài ý muốn về thủ tục Visa, đội tuyển Việt Nam đã không thể lên đường tới Romania tham dự vòng chung kết trực tiếp. Để tiếp tục đóng góp cho cộng đồng phát triển xe tự hành và những người đam mê công nghệ, chúng tôi quyết định mở kho tài liệu và mã nguồn này dưới dạng mã nguồn mở cho cộng đồng.

Bên cạnh đó, các mô hình xe tự hành dựa trên nền tảng này đã được thương mại hóa thành công cho các đơn vị giáo dục, nghiên cứu, mang lại doanh thu khoảng **$4,000 USD** để tái đầu tư và phát triển công nghệ.

---

## 🗺️ Kiến trúc hệ thống

```mermaid
graph TD
    %% Sensors to Input Nodes
    Cam[Camera Module] -->|Raw Video| CamNode[cameraNODE.py / Open CV]
    IMU_HW[BNO055 IMU via I2C] -->|Raw Orientation| IMUNode[imuNODE.py]
    Sonar_HW[Ultrasonic Sensors] -->|Echo Distance| SonarNode[sonarNODE.py / latead]
    GPS_HW[GPS Receiver] -->|Coordinates| GPSNode[gpstrackerNODE.py]

    %% Input Nodes to ROS topics
    CamNode -->|/camera/image_raw| ImageTopic[ROS: image_raw]
    IMUNode -->|/automobile/IMU| IMUTopic[ROS: imu]
    SonarNode -->|/automobile/sonar| SonarTopic[ROS: sonar]
    GPSNode -->|/automobile/localisation| LocalTopic[ROS: localisation]

    %% Brain/Decision Loop
    ImageTopic -->|OpenCV Frame| MainCar[main_car.py / Autonomous Loop]
    IMUTopic -->|State Feedback| MainCar
    SonarTopic -->|Obstacle Dist| MainCar
    LocalTopic -->|GPS Pos| MainCar

    %% Deep Learning Models in Main Car
    ONNX_Lane[lane_keeper_small.onnx] -->|e2, e3 errors| MainCar
    ONNX_Sign[sign_classifier.onnx] -->|Traffic Sign Class| MainCar
    ONNX_TL[trafficlight_classifier_small.onnx] -->|Light State| MainCar
    ONNX_Obs[pedestrian_classifier_small.onnx] -->|Obstacle Class| MainCar

    %% Decision to Output
    MainCar -->|Target Speed & Steer| Controller[controller3.py / Pure Pursuit / PID]
    Controller -->|Control Commands| DriveTopic[ROS: control cmd]
    
    %% Output to MCU
    DriveTopic -->|Serial Protocol| SerialNode[serialNODE.py]
    SerialNode -->|USB Serial /dev/ttyACM0| MCU[Arduino / Nucleo MCU]
    MCU -->|PWM Signals| Actuators[Motors & Steering Servo]
```

---

## 🔌 Sơ đồ kết nối phần cứng

Để xe có thể hoạt động chính xác, các cảm biến và mạch vi điều khiển cần được kết nối theo sơ đồ phần cứng dưới đây:

<p align="center">
  <img src="assets/Hardware_Connection.jpg" alt="Hardware Connections Diagram" width="800"/>
</p>

---

## 🏁 Đường chạy & Bản đồ thử nghiệm

Hệ thống được thử nghiệm trên sa bàn tiêu chuẩn của cuộc thi BFMC với bản đồ định vị GPS cục bộ và các vạch kẻ đường, biển báo:

<table>
  <tr>
    <td align="center"><b>Sa bàn thực tế</b></td>
    <td align="center"><b>Bản đồ định vị</b></td>
  </tr>
  <tr>
    <td><img src="assets/track.jpeg" alt="BFMC Test Track" width="450"/></td>
    <td><img src="assets/map.jpeg" alt="Localization Map" width="450"/></td>
  </tr>
</table>

---

## 📂 Cấu trúc thư mục dự án

```
Autopilot_and_FSD/
├── .catkin_workspace        # ROS catkin configuration file
├── LICENSE                  # License terms
├── README.md                # English system documentation
├── README_VN.md             # Vietnamese system documentation (file này)
├── network_conf_auto.bash   # Network configuration script
├── assets/                  # Images and diagrams
└── src/
    ├── action/              # High-level motion planning and maneuvers
    │   └── src/action/      # Maneuver implementations (overtaking, parking, etc.)
    ├── control/             # Low-level control interfaces and drivers
    │   └── src/control/     # Car data and state machine interfaces
    ├── input/               # Sensor interface nodes (Camera, IMU, GPS, Sonar, V2V)
    ├── output/              # MCU communication and serial interfaces (C++ / Python)
    ├── perception/          # Lane detection and computer vision package placeholders
    ├── rosserial_python/    # ROS standard rosserial communication protocol package
    ├── utils/               # Launch scripts, custom message/service definitions, requirements
    │   ├── launch/          # Launch configurations (*.launch)
    │   ├── msg/             # Custom message types (*.msg)
    │   ├── srv/             # Custom service types (*.srv)
    │   └── requirements.txt # Python dependency file
    └── tests/
        └── lane_keeping_6_cv_camera/   # Main autonomous driving and lane-keeping demo
            ├── models/                 # Pre-trained ONNX and PyTorch weights
            ├── main_car.py             # Main execution loop
            ├── detection.py            # Deep learning inference pipeline (ONNX Runtime)
            ├── controller3.py          # Steering (Pure Pursuit) and speed controller
            ├── maneuvers.py            # Local maneuver planner
            └── automobile_data.py      # ROS topic I/O manager
```

---

## 🛠️ Hướng dẫn cài đặt hệ thống

### 1. Tải & Ghi hệ điều hành Raspberry Pi OS
Tải xuống Raspberry Pi OS (Phiên bản Desktop hoặc Lite) từ trang chủ [Raspberry Pi Software](https://www.raspberrypi.com/software/operating-systems/). Ghi file image vào thẻ nhớ SD bằng phần mềm [Balena Etcher](https://www.balena.io/etcher/).

### 2. Thiết lập mạng và SSH
Tạo file `wpa_supplicant.conf` tại thư mục gốc của thẻ SD (phân vùng boot) để tự động kết nối Wi-Fi khi bật nguồn:

```ini
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=VN

network={
    ssid="SSID_WIFI_CỦA_BẠN"
    psk="MẬT_KHẨU_WIFI_CỦA_BẠN"
}
```

Tạo các file rỗng có tên `ssh`, `i2c`, và `camera` trong phân vùng boot của thẻ SD để kích hoạt các giao tiếp tương ứng.

---

### 3. Cài đặt ROS Noetic (trên hệ điều hành Raspberry Pi Buster)

```bash
# Thêm repository của ROS
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu buster main" > /etc/apt/sources.list.d/ros-noetic.list'

# Thêm GPG key của ROS
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# Cập nhật danh sách gói
sudo apt-get update && sudo apt-get upgrade -y

# Cài đặt các công cụ biên dịch
sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake python3-pip libopencv-dev python3-opencv

# Khởi tạo rosdep
sudo rosdep init
rosdep update
```

#### Tải & Biên dịch ROS Noetic (Bản rút gọn - Lite Version)
```bash
mkdir -p ~/ros_catkin_ws
cd ~/ros_catkin_ws

# Tạo danh sách gói ROS Lite
rosinstall_generator ros_comm sensor_msgs cv_bridge --rosdistro noetic --deps --wet-only --tar > noetic-ros_comm-wet.rosinstall 
wstool init src noetic-ros_comm-wet.rosinstall

# Tùy chọn: Tăng bộ nhớ RAM ảo (swap) lên 1GB để tránh lỗi thiếu bộ nhớ khi biên dịch
sudoedit /etc/dphys-swapfile  # Sửa CONF_SWAPSIZE=100 thành CONF_SWAPSIZE=1024
sudo dphys-swapfile swapoff && sudo dphys-swapfile setup && sudo dphys-swapfile swapon

# Biên dịch và cài đặt ROS
sudo src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/noetic -j1 -DPYTHON_EXECUTABLE=/usr/bin/python3
```

#### Tự động nạp môi trường ROS khi mở terminal
```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

### 4. Cài đặt các thư viện Python
Cài đặt các thư viện Python cần thiết thông qua file `requirements.txt` nằm trong thư mục `src/utils/`:

```bash
# Cài đặt các thư viện bổ trợ hệ thống
sudo apt install -y libatlas-base-dev

# Cài đặt thư viện Python
cd ~/Autopilot_and_FSD
pip3 install -r src/utils/requirements.txt
pip3 install numpy --upgrade
```

---

### 5. Thiết lập giao tiếp I2C cho IMU
Theo hướng dẫn thiết lập I2C từ kho lưu trữ [RTIMULib](https://github.com/RPi-Distro/RTIMULib/tree/master/Linux) để đảm bảo cảm biến BNO055 giao tiếp ổn định thông qua I2C.

---

## 🚀 Biên dịch & Chạy Demo

### 1. Biên dịch dự án
Sử dụng lệnh `catkin_make` để biên dịch toàn bộ Workspace:

```bash
cd ~/Autopilot_and_FSD
catkin_make
source devel/setup.bash
```

### 2. Khởi động các Node giao tiếp xe
Khởi chạy file launch để bật các node giao tiếp cảm biến và điều khiển từ xa:

```bash
roslaunch utils run_automobile_remote.launch
```

### 3. Chạy Demo tự lái giữ làn
Mở terminal mới, điều hướng đến thư mục demo và khởi chạy vòng lặp lái tự động:

```bash
cd ~/Autopilot_and_FSD/src/tests/lane_keeping_6_cv_camera
python3 main_car.py
```

---

## 🧠 Các mô hình học sâu trong hệ thống

Các mô hình được triển khai dưới dạng tệp tin `.onnx` trong thư mục `src/tests/lane_keeping_6_cv_camera/models/`:

| Tên mô hình | Định dạng | Nhiệm vụ |
| :--- | :--- | :--- |
| `lane_keeper_small.onnx` | ONNX | Ước lượng sai lệch quỹ đạo trái/phải ($e_2$) và sai số góc ($e_3$) so với tim đường để giữ làn. |
| `stop_line_estimator.onnx` | ONNX | Ước lượng khoảng cách từ xe tới vạch dừng trước mắt để chuẩn bị dừng ngã tư. |
| `local_path_estimator.onnx` | ONNX | Ước lượng chuỗi các điểm định vị cục bộ phía trước để dẫn hướng xe đi theo lộ trình. |
| `sign_classifier.onnx` | ONNX | Nhận diện 9 loại biển báo giao thông: `park`, `closed_road`, `highway_exit`, `highway_enter`, `stop`, `roundabout`, `priority`, `cross_walk`, `one_way`. |
| `trafficlight_classifier_small.onnx` | ONNX | Phát hiện và phân loại trạng thái đèn tín hiệu giao thông (Xanh, Vàng, Đỏ). |
| `pedestrian_classifier_small.onnx` | ONNX | Phân loại chướng ngại vật trước xe (người đi bộ `pedestrian`, rào chắn `roadblock`, xe khác). |

---

## 📝 Liên hệ & Bản quyền
Mọi bản quyền thuộc về **Duong Minh Ngoc Phat Corporation**. Vui lòng liên hệ với chúng tôi để biết thêm chi tiết.

*Copyright © 2024 Duong Minh Ngoc Phat Corporation. All Rights Reserved.*
