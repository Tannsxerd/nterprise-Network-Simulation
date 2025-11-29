# Enterprise Network Design Simulation

## Project Overview
project นี้จำลองการออกแบบ enterprise-Network ขนาดกลาง โดยใช้ความรู้ fundamental-network มาประยุกต์ใช้ เช่น Vlan , routing , DHCP(แจก ip) , และ config Nat เพื่อให้ enterprise สามารถออก internet ได้

## Network Topology
<img width="848" height="538" alt="image" src="https://github.com/user-attachments/assets/56412bce-7e5d-4323-9fe7-00dbfab1a9b5" />

## Key Features & Configurations

### 1. VLAN Segmentation
มีการแบ่ง VLAN เพื่อแยก Traffic ของแต่ละแผนกออกจากกัน ลดขนาด broad-cast เพิ่มความปลอดภัย Segmentation เพื่อลด surface และ จำกัดความเสียหายเมื่อถูกโจมตี:

- **VLAN 10 (HR):** แผนกบุคคล (192.168.10.0/24)  
- **VLAN 20 (Sales):** ฝ่ายขาย (192.168.20.0/24)  
- **VLAN 99 (IT/Admin):** แผนกIT/ผู้ดูแล (192.168.99.0/24)  

#### 🔧 Config Switch
**Create VLANs**
vlan 10
name HRdepartment

vlan 20
name SalesDepartmment

vlan 99
name IT/admin

Configure Access Ports (ต่อกับ PC หรือ Endpoint)
interface FastEthernet0/1
switchport mode access
switchport access vlan 10


Configure Trunk Port (ต่อกับ Enterprise-Router)
interface GigabitEthernet0/1
switchport mode trunk



2. Routing & Connectivity
- Router-on-a-Stick: ตั้งค่า sub-interface ของ router เพื่อให้ vlan สามารถสื่อสารกันได้ ผ่าน interface เดียว
- Static Routing: ตั้ง default route ของ enterprise-router เพื่อ route ไปยัง ISP
Physical Interface
interface GigabitEthernet0/0
no shutdown


Sub-interface for HR (VLAN 10)
interface GigabitEthernet0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0



3. IP Management (DHCP)
- ใช้งาน DHCP Server บน Enterprise-Router เพื่อแจก IP Address, Subnet Mask และ Default Gateway ให้อุปกรณ์ Endpoint แบบอัตโนมัติ
- ลดภาระ Admin ไม่ต้องไล่ config ทีละเครื่อง (Scalability) และป้องกันปัญหา IP Conflict (Human Error)
Config DHCP Server
ip dhcp pool HR_POOL
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1



4. Internet Access (NAT/PAT)
- Implemented NAT Overload (PAT) โดยการ Map Private IP จำนวนมากจากภายในองค์กร ให้สามารถออกไปใช้อินเทอร์เน็ตผ่าน Public IP เพียงตัวเดียว
- ใช้หมายเลข Port ในการแยก Session (Port Address Translation) ช่วยประหยัด Public IP Address และซ่อน Topology ภายในจากโลกภายนอก
Config NAT Overload ที่ Enterprise-Router
- กำหนดขา Inside (LAN) และ Outside (WAN)
interface GigabitEthernet0/0.10
ip nat inside

interface GigabitEthernet0/0.20
ip nat inside

interface GigabitEthernet0/1
ip nat outside


- สร้าง Access List (ACL)
access-list 1 permit 192.168.0.0 0.0.255.255


- สั่ง Map source list เข้ากับขา Outside โดยใช้ overload
ip nat inside source list 1 interface GigabitEthernet0/1 overload


- กำหนด Default Route
ip route 0.0.0.0 0.0.0.0 200.1.1.1



💻 Tech Stack
- Simulator: Cisco Packet Tracer
- Hardware Models: Cisco 2911 Router, Cisco 2960 Switch
- Protocols: TCP/IP, DHCP, NAT, 802.1Q (Trunking), ICMP

Verification
Ping Test to Internet (8.8.8.8):
Success reply from Google Server proving NAT configuration works.
source(HRpc) -> destination 8.8.8.8 (google dns server จำลอง)
<img width="866" height="357" alt="image" src="https://github.com/user-attachments/assets/430cb9eb-defb-428e-a352-a71bbd19a175" />
Ping Test cross Vlan:
source(HRpc) -> destination(salePC)
<img width="862" height="353" alt="image" src="https://github.com/user-attachments/assets/28c6e163-4532-4fad-aaf9-ecf8080ab340" />

Created by Punyaphat junpradub.
