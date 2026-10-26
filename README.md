# NAJA DRINKS — เว็บโปรโมทสินค้า (GitHub Pages)

## ไฟล์
| ไฟล์ | หน้าที่ |
|---|---|
| `index.html` | หน้าเว็บสำหรับลูกค้า |
| `admin.html` | หน้าหลังร้าน สำหรับแก้ข้อมูลร้าน เมนู ราคา และรูป (เข้าด้วย Google Login) |
| `admin-config.js` | URL ของ Admin API และ Google Client ID (ไม่ใช่ข้อมูลลับ) |
| `apps-script/Code.gs` | Admin API บน Google Apps Script (ไม่ได้ทำงานบน GitHub Pages ต้องนำไปวางใน Apps Script) |
| `data/products.json` | ข้อมูลทั้งหมดของเว็บ (หน้า admin แก้ไฟล์นี้ให้) |
| `pictures/` | รูปภาพ (รูปที่อัปโหลดจากหน้า admin จะอยู่ใน `pictures/uploads/`) |
| `.nojekyll` | บอก GitHub Pages ให้ส่งไฟล์ตามที่เป็นอยู่ ไม่ต้องแปลงด้วย Jekyll |

## 1. เอาเว็บขึ้น GitHub Pages
1. สมัครหรือล็อกอิน https://github.com แล้วกด **New repository**
   - Repository name: `naja` และเลือก **Public** (บัญชีฟรีเปิด Pages ได้เฉพาะ repo แบบ Public)
2. อัปโหลดไฟล์ทั้งหมดในโฟลเดอร์นี้ ด้วยวิธีใดวิธีหนึ่ง
   - **ผ่านเว็บ:** ในหน้า repo กด **Add file › Upload files** แล้วลากทั้งโฟลเดอร์ลงไป จากนั้นกด Commit
     (ถ้าไฟล์ `.nojekyll` ไม่ติดไปด้วย ให้กด Add file › Create new file ตั้งชื่อ `.nojekyll` แล้ว Commit เป็นไฟล์ว่าง)
   - **ผ่าน git:**
     ```bash
     git init -b main
     git add .
     git commit -m "first version"
     git remote add origin https://github.com/USERNAME/naja.git
     git push -u origin main
     ```
3. ไปที่ **Settings › Pages** แล้วตั้ง Source เป็น **Deploy from a branch**, Branch เป็น **main**, โฟลเดอร์เป็น **/ (root)** จากนั้นกด Save
4. รอประมาณ 1–2 นาที เว็บจะเปิดได้ที่ `https://USERNAME.github.io/naja/`
5. แก้ `USERNAME` ใน `index.html` บรรทัด `og:image` เป็นชื่อบัญชีของคุณ เพื่อให้มีรูปพรีวิวเวลาแชร์ลิงก์

## 2. ตั้งค่าหน้า admin (ทำครั้งเดียว)

ขั้นตอนการทำงาน: admin ล็อกอินด้วย Google แล้ว Apps Script ตรวจว่าอีเมลอยู่ในรายชื่อที่ได้รับสิทธิ์ ก่อนบันทึกข้อมูลลง GitHub
GitHub token จะเก็บไว้ใน Apps Script เท่านั้น ไม่ถูกส่งไปถึงเบราว์เซอร์ และ admin ไม่ต้องมีบัญชี GitHub

### 2.1 สร้าง GitHub token
1. เปิด https://github.com/settings/personal-access-tokens/new
   - Token name: `naja-admin`
   - Expiration: ตามต้องการ (เมื่อหมดอายุให้สร้างใหม่ แล้วนำไปแก้ใน Script properties)
   - Repository access: **Only select repositories** แล้วเลือก `naja`
   - Permissions › Repository permissions › **Contents: Read and write**
2. กด **Generate token** แล้วคัดลอกเก็บไว้ (GitHub จะแสดงให้ดูแค่ครั้งเดียว)

### 2.2 สร้าง Google Client ID (สำหรับปุ่ม Google Login)
1. เปิด https://console.cloud.google.com แล้วสร้างโปรเจกต์ใหม่ เช่น `naja-admin`
2. ไปที่ **Google Auth Platform › Branding** (หรือ OAuth consent screen) กรอกชื่อแอปและอีเมล แล้วเลือก Audience เป็น **External**
3. ไปที่ **Clients › Create client**
   - Application type: **Web application**
   - Authorized JavaScript origins: `https://USERNAME.github.io` (ถ้าจะทดสอบในเครื่อง ให้เพิ่ม `http://localhost:8000` ด้วย)
4. คัดลอก **Client ID** (ลงท้ายด้วย `.apps.googleusercontent.com`)

### 2.3 สร้าง Admin API บน Apps Script
1. เปิด https://script.google.com แล้วกด **New project** ตั้งชื่อ `NAJA Admin API`
2. วางโค้ดจาก `apps-script/Code.gs`
3. ไปที่ **⚙️ Project Settings › Script properties** แล้วเพิ่มค่าต่อไปนี้

   | Property | ค่า |
   |---|---|
   | `GITHUB_TOKEN` | token จากข้อ 2.1 |
   | `GITHUB_OWNER` | ชื่อบัญชี GitHub |
   | `GITHUB_REPO` | `naja` |
   | `GITHUB_BRANCH` | `main` |
   | `GOOGLE_CLIENT_ID` | Client ID จากข้อ 2.2 |
   | `ADMIN_EMAILS` | อีเมลที่เป็น admin คั่นด้วยจุลภาค เช่น `owner@gmail.com, staff@gmail.com` |

4. กลับไปที่หน้าโค้ด เลือกฟังก์ชัน `testSetup` แล้วกด **Run** (ครั้งแรกต้องกดอนุญาตสิทธิ์) ถ้าตั้งค่าถูก log จะขึ้นว่า "เชื่อม GitHub ได้"
5. **Deploy › New deployment › Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
6. คัดลอก **Web app URL** (ลงท้ายด้วย `/exec`)

### 2.4 ใส่ค่าใน `admin-config.js`
```js
window.ADMIN_CONFIG = {
  apiUrl: 'https://script.google.com/macros/s/....../exec',   // จากข้อ 2.3
  googleClientId: '......apps.googleusercontent.com'          // จากข้อ 2.2
};
```
Commit ไฟล์นี้ขึ้น GitHub แล้วเปิด `https://USERNAME.github.io/naja/admin.html` จากนั้นกด **Sign in with Google**

### เพิ่มหรือลบ admin
แก้ `ADMIN_EMAILS` ใน Apps Script › Project Settings › Script properties แล้วกด Save ใช้ได้ทันที ไม่ต้อง deploy ใหม่

## 3. แก้ข้อมูลประจำวัน
1. เปิด `admin.html` แล้วแก้ข้อมูลร้าน ราคา เมนู หรือรูป
   - ปุ่ม **แสดงบนเว็บ** ใช้ซ่อนเมนูที่หมดชั่วคราว
   - ปุ่ม **↑ ↓** ใช้สลับลำดับเมนู
   - ปุ่ม **📷** อัปโหลดรูป ระบบจะย่อรูปให้อัตโนมัติ
2. กด **💾 บันทึก & เผยแพร่**
3. หน้าเว็บจะอัปเดตภายในประมาณ 1–2 นาที

ทุกครั้งที่บันทึก GitHub จะเก็บประวัติไว้ ถ้าแก้ผิดสามารถดูหรือย้อนกลับได้ที่หน้า repo › `data/products.json` › **History**

## ความปลอดภัย
- ใครก็เปิดหน้า `admin.html` ได้ แต่ทุกคำสั่งที่ส่งไป Apps Script ต้องแนบ Google ID token และ Apps Script จะตรวจกับ Google ทุกครั้งว่าเป็นอีเมลในรายชื่อ `ADMIN_EMAILS` จริง
- ล็อกอินหนึ่งครั้งใช้งานได้ 1 ชั่วโมง เมื่อหมดเวลาจะมีปุ่ม Sign in ขึ้นที่แถบด้านบน ข้อมูลที่แก้ค้างไว้จะไม่หาย
- GitHub token อยู่ใน Script properties เท่านั้น **ห้ามใส่ token ลงในไฟล์ของ repo** เพราะ repo เป็นแบบ Public
- ถ้าสงสัยว่า token หลุด ให้ลบที่ https://github.com/settings/personal-access-tokens แล้วสร้างใหม่ จากนั้นนำไปแก้ใน Script properties
- ประวัติการบันทึกบน GitHub จะระบุอีเมลของ admin ที่แก้ไขในแต่ละครั้ง

## ทดสอบในเครื่อง
หน้าเว็บโหลดข้อมูลจาก `data/products.json` จึงต้องเปิดผ่าน local server (ดับเบิลคลิกไฟล์ตรงๆ ไม่ได้)
```bash
python -m http.server 8000
```
จากนั้นเปิด http://localhost:8000
