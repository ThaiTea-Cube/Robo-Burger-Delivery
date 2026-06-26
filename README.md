# 🍔 หน้าเว็บสั่งการหุ่นยนต์ส่งเบอร์เกอร์

หน้าเว็บไฟล์เดียว (`index.html`) สำหรับสั่งการหุ่นยนต์ผ่าน ROS 2
มี 2 ปุ่ม: **ไปจุด A** และ **กลับ Home**

## ทำงานยังไง

```
[index.html + roslibjs]  --WebSocket-->  [rosbridge_server]  --ROS 2-->  /web_command (std_msgs/String)
       กดปุ่ม                ws://IP:9090       (ฝั่ง ROS 2)          ↓
                                                          [Python Commander Node]  →  Nav2 / cmd_vel
```

- ปุ่ม **ไปจุด A** → publish ข้อความ `"GO_A"`
- ปุ่ม **กลับ Home** → publish ข้อความ `"GO_HOME"`
- ส่งไปที่ topic `/web_command` ชนิด `std_msgs/String`

> หน้าเว็บนี้รู้แค่การส่งคำสั่ง String — เรื่องพิกัด/การนำทางเป็นหน้าที่ของ Python Commander Node (ข้อ 2)

## วิธีใช้

### 1) ติดตั้งและรัน rosbridge ฝั่ง ROS 2 (บนเครื่องหุ่นยนต์/คอม ROS)

```bash
sudo apt install ros-$ROS_DISTRO-rosbridge-suite
ros2 launch rosbridge_server rosbridge_websocket_launch.xml
# จะเปิด WebSocket server ที่พอร์ต 9090
```

### 2) เปิดหน้าเว็บ

ดับเบิลคลิก `index.html` หรือเปิดด้วยเบราว์เซอร์

- ถ้าเปิดเว็บบนเครื่องเดียวกับ ROS → ใช้ค่าเริ่มต้น `ws://localhost:9090`
- ถ้าเปิดบนมือถือ/เครื่องอื่น → เปลี่ยนเป็น IP ของเครื่อง ROS เช่น `ws://192.168.1.50:9090`

กด **เชื่อมต่อ** → ไฟสถานะเป็นสีเขียว → กดปุ่มสั่งงานได้

### 3) ทดสอบว่าคำสั่งส่งถึง ROS จริง

เปิดอีกหน้าต่าง terminal บนเครื่อง ROS แล้วดู topic:

```bash
ros2 topic echo /web_command
```

กดปุ่มบนเว็บแล้วจะเห็น `data: GO_A` หรือ `data: GO_HOME` ขึ้นมา

## การปรับแต่ง

แก้ค่าด้านบนสุดของ `<script>` ใน `index.html`:

```js
const COMMAND_TOPIC = '/web_command';     // ชื่อ topic
const COMMAND_TYPE  = 'std_msgs/String';  // ชนิดข้อความ
```

## ส่วนที่ยังต้องทำต่อ (นอกเหนือจากหน้าเว็บ)

- [ ] **Python Commander Node** (ข้อ 2): subscribe `/web_command` → สั่ง Nav2 ไปพิกัด A / Home
- [ ] **micro-ROS** (ข้อ 3): subscribe `/cmd_vel` → แปลงเป็น PWM ขับมอเตอร์
