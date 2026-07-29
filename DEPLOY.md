# คู่มือนำเว็บขึ้นออนไลน์แบบมีโดเมนของตัวเอง

โฟลเดอร์นี้ (`d:\wedding`) ถูกเตรียมเป็น git repository ไว้แล้ว (ดูท้ายไฟล์) พร้อม push ขึ้น GitHub แล้วต่อยอดไปมี URL จริงของตัวเอง เช่น `thakoon-kantarat.com` ทำตามลำดับนี้ได้เลย

## 1. สร้างบัญชี GitHub (ถ้ายังไม่มี)
ไปที่ https://github.com/signup สมัครฟรี

## 2. สร้าง repository ใหม่บน GitHub
- กด "New repository" ตั้งชื่อ เช่น `thakoon-kantarat-wedding`
- เลือก Public หรือ Private ก็ได้ (Private ก็ deploy ได้ปกติ)
- **ไม่ต้อง** ติ๊ก "Add a README" (เรามีไฟล์อยู่แล้ว)
- กด Create repository แล้วคัดลอก URL ของ repo ที่ได้ (เช่น `https://github.com/USERNAME/thakoon-kantarat-wedding.git`)

## 3. Push โค้ดขึ้น GitHub
เปิด terminal ที่โฟลเดอร์ `d:\wedding` แล้วรัน (แทน URL ด้วยของจริงจากข้อ 2):

```bash
git remote add origin https://github.com/USERNAME/thakoon-kantarat-wedding.git
git branch -M main
git push -u origin main
```

## 4. Deploy แบบฟรีด้วย Netlify (ไม่ต้อง build ใดๆ เพราะเป็น static HTML ล้วน)
1. ไปที่ https://app.netlify.com/signup สมัคร (แนะนำให้ "Sign up with GitHub" จะได้เชื่อมง่าย)
2. กด "Add new site" → "Import an existing project" → เลือก GitHub → เลือก repo `thakoon-kantarat-wedding`
3. ช่อง Build command เว้นว่างไว้, Publish directory ใส่ `.` (จุดเดียว เพราะ index.html อยู่ที่ root)
4. กด Deploy — ไม่กี่วินาทีจะได้ลิงก์แบบ `https://random-name-12345.netlify.app` ใช้เปิดทดสอบได้ทันที

## 5. ซื้อโดเมนของตัวเอง
เลือกผู้ให้บริการที่สะดวก เช่น:
- Cloudflare Registrar (https://domains.cloudflare.com) — ราคาต้นทุนไม่บวกกำไร
- Namecheap (https://www.namecheap.com)
ค้นหาชื่อที่ต้องการ เช่น `thakoonkantarat.com` หรือ `tk-wedding.love` แล้วซื้อ (ปกติราคา ~300-500 บาท/ปี)

## 6. ผูกโดเมนเข้ากับ Netlify
1. ในหน้า site บน Netlify ไปที่ "Domain settings" → "Add a domain"
2. พิมพ์โดเมนที่ซื้อมา Netlify จะโชว์ค่า DNS ที่ต้องตั้ง (ปกติเป็น CNAME หรือ A record ชี้ไปที่ Netlify)
3. เอาค่านั้นไปกรอกในหน้า DNS ของผู้ให้บริการโดเมน (ผู้ให้บริการแต่ละเจ้าหน้าตาไม่เหมือนกัน แต่จะมีเมนู "DNS records"/"DNS management")
4. รอ 10 นาที - ไม่กี่ชั่วโมงให้ DNS อัปเดต Netlify จะออก SSL (https) ให้อัตโนมัติ

## หลังจากนั้น
ทุกครั้งที่แก้ไข `index.html` แล้ว `git push` ใหม่ Netlify จะ deploy เวอร์ชันล่าสุดให้อัตโนมัติภายในไม่กี่วินาที ไม่ต้องทำซ้ำขั้นตอนด้านบนอีก

---

*หมายเหตุ: repo นี้ยังไม่ได้ push ขึ้น GitHub จริง (ทำได้แค่ `git init` + commit ในเครื่องนี้เท่านั้น เพราะต้องใช้บัญชี GitHub ของคุณเอง) ทำตามข้อ 1-3 ด้านบนเพื่อ push ขึ้นจริง*
