# รายงานสรุปสมาชิกสมาคม · Thai Future Food

เว็บรายงานทะเบียนสมาชิกสมาคม ธีมสีเหลือง-ชมพู อ่านข้อมูลจาก Google Sheets ใน Google Drive
เข้าใช้ได้เฉพาะบัญชี Google **admin@thaifuturefood.org**

## ทำอะไรได้
- แสดงข้อมูลสมาชิกแต่ละราย: ชื่อบริษัท (ไทย/อังกฤษ), ผู้แทน 1 และ 2, ที่อยู่สำนักงาน พร้อมระบุว่าเป็นสำนักงานใหญ่หรือสาขา, ประเภทสมาชิก (สามัญ/วิสามัญ), ประเภทวิสาหกิจ (S/M/L), ผู้ติดต่อ เบอร์โทร อีเมล เว็บไซต์
- สรุปยอดสมาชิก และกราฟตามประเภทวิสาหกิจ ลักษณะธุรกิจ และจังหวัด
- ค้นหาและกรองตามประเภทสมาชิก ประเภทวิสาหกิจ สำนักงานใหญ่/สาขา และจังหวัด
- คัดลอกอีเมลหรือตารางของรายชื่อที่กรองไว้
- หน้า `#pitch` สำหรับนำเสนอ 1 นาที ตอบ 3 ข้อ พร้อมนาฬิกาจับเวลา

## ทำงานอย่างไร
| ส่วน | ใช้อะไร |
|---|---|
| โฮสต์ | GitHub Pages (`index.html`) |
| เข้าสู่ระบบ | Google Sign-In (Google Identity Services) ตรวจว่าอีเมลเป็น `admin@thaifuturefood.org` |
| ฐานข้อมูล | Google Sheet "TFFA ทะเบียนสมาชิกสมาคม (ข้อมูลเว็บรายงาน)" ชีต `Members` ใน Drive ของ admin |
| ดึงข้อมูล | Google Sheets API (อ่านอย่างเดียว) ด้วยสิทธิ์ของคนที่เข้าสู่ระบบ รีเฟรชทุก 5 นาที |

ป้องกันไว้ 2 ชั้น:
1. หน้าเว็บตรวจอีเมลที่ Google ยืนยันแล้ว ถ้าไม่ตรงจะยกเลิกสิทธิ์ทันที
2. Google เองไม่ยอมให้บัญชีที่ไม่มีสิทธิ์ในไฟล์อ่าน Sheet

ข้อมูลสมาชิก**ไม่ได้**อยู่ใน repo นี้ และไม่ได้ฝังในหน้าเว็บ

## ตั้งค่าครั้งแรก (ทำครั้งเดียว ประมาณ 10 นาที)

### 1. เปิด GitHub Pages
Settings › Pages › Build and deployment › Source: **Deploy from a branch** › Branch: **main** / **(root)** › Save
จะได้ URL `https://admin-tfa.github.io/tffa-member-report/`

> GitHub Pages ของ repo แบบ private ใช้ได้เฉพาะแพ็กเกจ GitHub แบบเสียเงิน (Pro/Team) ถ้าเป็นบัญชีฟรี repo ต้องเป็น public ซึ่งยังปลอดภัย เพราะใน repo ไม่มีข้อมูลสมาชิก

### 2. สร้าง OAuth Client ID ใน Google Cloud (ล็อกอินด้วย admin@thaifuturefood.org)
1. ไปที่ https://console.cloud.google.com/ › สร้างโปรเจกต์ใหม่ เช่น `tffa-member-report`
2. **APIs & Services › Library** › เปิดใช้ **Google Sheets API**
3. **APIs & Services › OAuth consent screen** (หรือ Google Auth Platform › Branding/Audience)
   - User type: **Internal** (ใช้ได้เพราะ thaifuturefood.org เป็น Google Workspace และไม่ต้องผ่านการตรวจสอบของ Google)
   - ใส่ชื่อแอป เช่น "TFFA Member Report" และอีเมลติดต่อ
   - Scopes: `openid`, `email`, `.../auth/spreadsheets.readonly`
4. **APIs & Services › Credentials › Create credentials › OAuth client ID**
   - Application type: **Web application**
   - Authorized JavaScript origins: `https://admin-tfa.github.io`
     (ถ้าทดสอบในเครื่อง เพิ่ม `http://localhost:8000`)
   - ไม่ต้องใส่ Redirect URI
5. คัดลอก **Client ID** (ลงท้ายด้วย `.apps.googleusercontent.com`)

### 3. ใส่ Client ID ใน `config.js`
```js
window.TFFA_CONFIG = {
  clientId: '1234567890-xxxx.apps.googleusercontent.com'
};
```
แล้ว commit ขึ้น `main` จากนั้นรอ GitHub Pages อัปเดตประมาณ 1 นาที
Client ID เผยแพร่ได้ แต่**ห้ามใส่ Client Secret** ในไฟล์นี้

### ทดสอบในเครื่อง
```bash
python3 -m http.server 8000
# เปิด http://localhost:8000
```

## ไฟล์
- `index.html` — เว็บสำหรับ GitHub Pages (Google Sign-In + Sheets API)
- `config.js` — ตั้งค่า Google OAuth Client ID
- `claude-artifact.html` — เวอร์ชันที่รันใน claude.ai (ใช้ Claude connector และตรวจบัญชี Claude)

## โครงสร้างคอลัมน์ในชีต `Members`
เลขที่ · ชื่อบริษัท (ไทย) · ชื่อบริษัท (อังกฤษ) · เลขทะเบียนนิติบุคคล · ผู้แทน 1 · ผู้แทน 1 (EN) · ผู้แทน 2 · ผู้แทน 2 (EN) · ที่อยู่สำนักงาน · สำนักงาน · จังหวัด · ประเภทสมาชิก · ประเภทวิสาหกิจ · ลักษณะธุรกิจ · ผู้ติดต่อ · โทรศัพท์ · โทรสาร · อีเมล · เว็บไซต์ · วันเริ่มสมาชิกภาพ

ถ้าเพิ่มหรือแก้สมาชิกใน Sheet เว็บจะอัปเดตเองภายใน 5 นาที ให้คงชื่อหัวคอลัมน์ไว้ตามนี้
