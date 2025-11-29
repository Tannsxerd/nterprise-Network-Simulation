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
  
!Config Switch 
! Create VLANs
vlan 10
name HRdepartment
vlan 20
name SalesDepartmment
vlan 99
name IT/admin

! Configure Access Ports Vlan10,20,99 (switch ต่อกับ pc หรือ endpoint)
Example:
interface FastEthernet0/1
switchport mode access
switchport access vlan 10

! Configure Trunk Port (switch ต่อกับ enterprise-router)
เพื่อให้ Vlan สามารถคุยกันได้ โดยการใส่ VLAN Tag (IEEE 802.1Q) ของแต่ vlan เข้าไป ทำให้รู้ว่า frame ไหน มาจาก vlan ไหน และ ลดจำนวนสายที่ต้องใช้
Example:
interface GigabitEthernet0/1
switchport mode trunk
_____________________________________________________________________________________________________________________________________________________________________________________________________________________

### 2. Routing & Connectivity
- **Router-on-a-Stick:** ตั้งค่า sub-interface ของ router เพื่อให้ vlan สามารถสื่อสารกันได้ ผ่าน interface เดียว.
- **Static Routing:**  ตั้ง defalut route ของ enterprise-router  เพื่อ route ไปยัง ISP.
! Physical Interface
interface GigabitEthernet0/0
no shutdown
! Configure Router-on-a-Stick
Ex.config HR 
! Sub-interface for HR (VLAN 10)
interface GigabitEthernet0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
_____________________________________________________________________________________________________________________________________________________________________________________________________________________


### 3. IP Management (DHCP)
- ใช้งาน DHCP Server บน Enterprise-Router เพื่อแจก IP Address, Subnet Mask และ Default Gateway ให้อุปกรณ์ Endpoint แบบอัตโนมัติ
  ช่วยลดภาระ Admin ไม่ต้องไล่ config ทีละเครื่อง (Scalability) และป้องกันปัญหา IP Conflict (Human Error):

! Config DHCP Server 
! สร้าง DHCP Pool สำหรับแต่ละแผนก 
Example: ip dhcp pool HR_POOL network 192.168.10.0 255.255.255.0 
!กำหนด gate-way ที่จะออกนอก enterprise-router ของ network 192.168.10.0 (HRdepartment)
default-router 192.168.10.1!
____________________________________________________________________________________________________________________________________________________________________________________________________________________

### 4. Internet Access (NAT/PAT)
- Implemented NAT Overload (PAT) โดยการ Map Private IP จำนวนมากจากภายในองค์กร ให้สามารถออกไปใช้อินเทอร์เน็ตผ่าน Public IP เพียงตัวเดียว
  โดยใช้หมายเลข Port ในการแยก Session (Port Address Translation) ช่วยประหยัด Public IP Address และซ่อน Topology ภายในจากโลกภายนอก (Security through obscurity):

! Config NAT Overload ที่  enterprise-router 
1. กำหนดขา Inside (ฝั่ง LAN) และ Outside (ฝั่ง WAN) 
ต้องเข้าไปกำหนดใน Sub-interface ทุกอันที่เป็นวงภายใน 
Example: 
interface GigabitEthernet0/0.10 
ip nat inside 
interface GigabitEthernet0/0.20 
ip nat inside

2. กำหนดขาที่ต่อออกไปยัง ISP เป็น Outside 
interface GigabitEthernet0/1 ip nat outside

3. สร้าง Access List (ACL) เพื่อระบุ Scope ว่า IP วงไหนได้รับอนุญาตให้ทำ NAT บ้าง
access-list 1 permit 192.168.0.0 0.0.255.255 **ง่ายๆคือ 192.168.x.x**

4. สั่ง Map source list เข้ากับขา Outside โดยใช้คำสั่ง overload
ip nat inside source list 1 interface GigabitEthernet0/1 overload

5. กำหนด Default Route (ชี้ทางออกไปหา Router ISP เพื่อให้ Traffic ที่ไม่รู้จักส่งออกไปข้างนอก)
ip route 0.0.0.0 0.0.0.0 200.1.1.1
____________________________________________________________________________________________________________________________________________________________________________________________________________________

## 💻 Tech Stack
- **Simulator:** Cisco Packet Tracer
- **Hardware Models:** Cisco 2911 Router, Cisco 2960 Switch
- **Protocols:** TCP/IP, DHCP, NAT, 802.1Q (Trunking), ICMP
____________________________________________________________________________________________________________________________________________________________________________________________________________________

## Verification
**Ping Test to Internet (8.8.8.8):**
Success reply from Google Server proving NAT configuration works.
source(HRpc) -> destination 8.8.8.8(google dns server จำลอง)
<img width="866" height="357" alt="image" src="https://github.com/user-attachments/assets/430cb9eb-defb-428e-a352-a71bbd19a175" />
**Ping Test cross Vlan):**
source(HRpc) -> destination(salePC)
<img width="862" height="353" alt="image" src="https://github.com/user-attachments/assets/28c6e163-4532-4fad-aaf9-ecf8080ab340" />
---

*Created by Punyaphat junpradub.*
