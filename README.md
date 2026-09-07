# ระบบ E-Approval ขออนุญาตไปราชการ | โรงเรียนวัดบางปูน

ระบบบริหารจัดการและขออนุมัติเดินทางไปราชการอิเล็กทรอนิกส์ (E-Approval) สำหรับ **โรงเรียนวัดบางปูน** ออกแบบตามมาตรฐานงานสารบรรณราชการไทย ใช้งานง่าย รองรับการแสดงผลทุกอุปกรณ์ (Responsive ทั้งมือถือ แท็บเล็ต และคอมพิวเตอร์) พร้อมระบบสั่งพิมพ์ออกเป็นเอกสาร **"บันทึกข้อความ"** ขนาด A4 ที่ได้มาตรฐานทางราชการ 100%

---

## 🌟 จุดเด่นของระบบ (Key Highlights)

1. **ใช้งานได้ทันที (Zero Installation)**: เขียนด้วย Vanilla HTML5, Tailwind CSS, และ JavaScript แบบ Standalone Single-file สามารถดับเบิ้ลคลิกเปิดไฟล์ `index.html` ใน Google Chrome, Microsoft Edge, Safari ได้ทันที ไม่ต้องติดตั้งโปรแกรมหรือตั้งค่า Web Server
2. **ระบบพิมพ์บันทึกข้อความราชการมาตรฐาน A4 เป๊ะ 100%**:
   - ตราครุฑราชการคมชัด (Vector SVG ความสูงตามระเบียบสารบรรณ 1.5 - 3 ซม.)
   - ฟอนต์ภาษาไทย **TH Sarabun / Sarabun 16pt** ระยะบรรทัดและย่อหน้าตรงตามระเบียบงานสารบรรณ พ.ศ. 2526
   - เครื่องหมายเช็คถูก `[✓]` อัตโนมัติในช่องตัวเลือกงบประมาณและยานพาหนะตามที่ครูกรอกจริง
   - ประทับตรา **ลายเซ็นอิเล็กทรอนิกส์ (E-Signature Stamp)** ของผู้อำนวยการโรงเรียนและวันที่อนุมัติอัตโนมัติ
   - กรอง UI และปุ่มกดออกทั้งหมดในโหมดสั่งพิมพ์ (`@media print`)
3. **ระบบจำลองฐานข้อมูล Relational Database (LocalStorage)**:
   - ตาราง `tb_users` และ `tb_requests`
   - มีชุดข้อมูลตัวอย่างเริ่มต้น (Pre-seeded Mock Data) ให้ทดลองใช้งานได้ทันที
   - รองรับการสร้างคำขอใหม่, ตรวจสอบสถานะ, และการอนุมัติ/ไม่อนุมัติ
4. **โหมดทดสอบรวดเร็ว (Demo Switcher Toolbar)**:
   - แถบเครื่องมือด้านบนสุด สามารถกดสลับสิทธิ์ระหว่าง **ครู (Teacher)** และ **ผู้อำนวยการ (Director)** ได้ด้วย 1 คลิก

---

## 👥 บัญชีผู้ใช้งานสำหรับทดสอบ (Demo Accounts)

| ชื่อผู้ใช้ (Username) | รหัสผ่าน (Password) | ชื่อ-นามสกุล | ตำแหน่ง | บทบาท (Role) |
| :--- | :--- | :--- | :--- | :--- |
| `teacher1` | `1234` | นายสมชาย ใจดี | ครู ชำนาญการ | ครูผู้ขอ (Teacher) |
| `teacher2` | `1234` | นางสาวสายใจ ใฝ่เรียน | ครู คศ.๑ | ครูผู้ขอ (Teacher) |
| `director` | `1234` | นายวิชัย เก่งการศึกษา | ผู้อำนวยการโรงเรียนวัดบางปูน | ผู้อำนวยการ (Director) |

*(หมายเหตุ: บนหน้าเว็บมีปุ่ม Quick Switcher ด้านบนสุด สามารถกดสลับบัญชีได้ทันทีโดยไม่ต้องกรอกรหัสผ่าน)*

---

## 📁 โครงสร้างตารางฐานข้อมูล (Database Schema)

### 1. ตารางผู้ใช้ `tb_users`
```sql
CREATE TABLE tb_users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    fullname VARCHAR(150) NOT NULL,
    position VARCHAR(100) NOT NULL,
    role ENUM('teacher', 'director') NOT NULL DEFAULT 'teacher',
    signature_url TEXT NULL
);
```

### 2. ตารางคำขอไปราชการ `tb_requests`
```sql
CREATE TABLE tb_requests (
    request_id VARCHAR(20) PRIMARY KEY,     -- เช่น REQ-2569-001
    user_id INT NOT NULL,                   -- FK อ้างอิง tb_users.id
    doc_number VARCHAR(50) NOT NULL,        -- เช่น วป ๐๑/๒๕๖๙
    subject VARCHAR(255) NOT NULL,          -- เรื่องขออนุมัติไปราชการ
    write_date DATE NOT NULL,               -- วันที่เขียนบันทึก
    start_date DATE NOT NULL,               -- วันที่เริ่มต้นไปราชการ
    end_date DATE NOT NULL,                 -- วันที่สิ้นสุด
    location VARCHAR(255) NOT NULL,         -- สถานที่ไปราชการ
    details TEXT NOT NULL,                  -- รายละเอียด/อ้างถึงหนังสือราชการ
    expense_type ENUM('none', 'full', 'partial') NOT NULL,
    expense_details JSON NULL,              -- { allowance: true, transport: true, accommodation: false }
    travel_method ENUM('gov_car', 'own_car', 'bike', 'public') NOT NULL,
    vehicle_plate VARCHAR(50) NULL,         -- ทะเบียนรถ (กรณีใช้รถส่วนบุคคล/มอเตอร์ไซค์)
    status ENUM('pending', 'approved', 'rejected') NOT NULL DEFAULT 'pending',
    director_comment TEXT NULL,             -- ความเห็น/ข้อสั่งการของผู้อำนวยการ
    approved_at DATETIME NULL,              -- วันเวลาที่อนุมัติ
    created_at DATETIME NOT NULL,
    FOREIGN KEY (user_id) REFERENCES tb_users(id)
);
```

---

## 🚀 ขั้นตอนการติดตั้งและการเปิดใช้งาน

### วิธีที่ 1: เปิดใช้งานทันที (Offline / Standalone)
1. ดับเบิ้ลคลิกที่ไฟล์ `index.html` เพื่อเปิดผ่านเว็บเบราว์เซอร์ (Chrome / Edge / Safari / Firefox)
2. ระบบจะโหลดข้อมูลเริ่มต้นและเข้าสู่หน้าจอของ **ครูสมชาย** ให้อัตโนมัติ
3. สามารถทดลองสร้างคำขอใหม่, สลับเป็นผู้อำนวยการเพื่อกดอนุมัติ, และกดปุ่ม **"พิมพ์"** เพื่อออกเอกสาร A4

### วิธีที่ 2: รันผ่าน Local Web Server (แนะนำสำหรับการทดสอบเครือข่ายภายในโรงเรียน)
หากต้องการแชร์ให้ครูในโรงเรียนเปิดผ่านมือถือ/Wi-Fi เดียวกัน:
```powershell
# ใช้ Python สร้าง HTTP Server ชั่วคราว
python -m http.server 8080
```
เปิดเบราว์เซอร์ไปที่: `http://localhost:8080` (หรือ `http://<IP_เครื่องคอมพิวเตอร์>:8080` บนมือถือ)

---

## 🖨️ คำแนะนำสำหรับการสั่งพิมพ์เอกสาร A4 (Print to PDF / Paper)

1. เมื่อคำขอมีสถานะ **"อนุมัติแล้ว"** จะปรากฏปุ่ม **"พิมพ์"**
2. เมื่อกดปุ่ม ระบบจะเปิดหน้าต่าง Print Preview ของเบราว์เซอร์อัตโนมัติ
3. **การตั้งค่าในหน้าต่างพิมพ์ (Print Dialog)**:
   - **Destination**: บันทึกเป็น PDF (Save as PDF) หรือเลือกเครื่องพิมพ์ A4
   - **Pages**: ทั้งหมด (All) - เอกสารจะถูกจัดให้อยู่ใน 1 หน้าพอดี
   - **Layout**: แนวตั้ง (Portrait)
   - **Paper Size**: A4
   - **Margins**: ค่าเริ่มต้น (Default) หรือ ไม่มีขอบ (None) เนื่องจากระบบได้ตั้งขอบกระดาษ 1.5 - 2.5 ซม. ตามระเบียบสารบรรณไว้แล้ว
   - **Options**: ติ๊กถูกที่ **"Background graphics" (กราฟิกพื้นหลัง)** เพื่อให้แสดงผลแถบเส้นและสีได้อย่างสมบูรณ์

---

## 🔌 แนวทางการเชื่อมต่อ Backend จริงในอนาคต

### ตัวเลือก A: Google Apps Script + Google Sheets (ฟรี 100% ดูแลรักษาง่ายที่สุดสำหรับโรงเรียน)
1. สร้าง Google Sheet มีแท็บ `tb_users` และ `tb_requests`
2. สร้าง Google Apps Script เป็น Web App (doGet / doPost)
3. ปรับฟังก์ชัน `getRequests()` และ `saveRequests()` ใน JavaScript ให้เรียกผ่าน `fetch('https://script.google.com/macros/s/.../exec')`

### ตัวเลือก B: PHP + MySQL (สำหรับรันบน Web Hosting ของโรงเรียน)
1. นำสคริปต์ SQL ด้านบนไปสร้างใน MySQL / phpMyAdmin
2. สร้างไฟล์ `api.php` เพื่อรับส่ง JSON ผ่าน RESTful API:
   - `GET /api.php?action=get_requests`
   - `POST /api.php?action=create_request`
   - `POST /api.php?action=approve_request`
3. เชื่อมต่อระบบแจ้งเตือนผ่าน **LINE Notify** ไปยังกลุ่มครูหรือผู้อำนวยการเมื่อมีคำขอใหม่
