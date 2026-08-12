# วิธีตั้งค่า Supabase สำหรับ RSVP + Dashboard

> **สถานะ: ตั้งค่าเสร็จแล้ว ✅** โปรเจกต์จริงที่ใช้งานอยู่คือ `wedding-rsvp-v2` (`https://rusyltzeomupisfdhtvr.supabase.co`) เชื่อมเข้ากับ `index.html` และ `dashboard.html` เรียบร้อยแล้ว ทดสอบส่ง/แก้ไข RSVP จริงผ่านแล้ว เอกสารนี้เก็บไว้อ้างอิง เผื่อวันหลังต้องสร้างโปรเจกต์ใหม่หรือมีปัญหาต้องแก้ไข

ระบบ RSVP ใช้ฐานข้อมูลของเราเอง (Supabase — Postgres ฟรี) แขกกรอกคำตอบได้ และถ้าแก้ไขคำตอบเดิมก็ทำได้จริง (ดูกลไกด้านล่าง) พร้อมหน้า Dashboard ให้คู่บ่าวสาวดูผลได้สวยๆ

## ขั้นตอนตั้งค่า (ถ้าต้องทำใหม่)

### 1. สมัครบัญชีและสร้างโปรเจกต์
1. ไปที่ https://supabase.com → "Start your project" → สมัครฟรี
2. "New project" ตั้งชื่อ → ตั้งรหัสผ่านฐานข้อมูล → เลือก Region ใกล้ไทย (Singapore)
3. **ตอนสร้างโปรเจกต์**: ปิดตัวเลือก **"Automatically expose new tables"** และ **"Enable automatic RLS"** (เราตั้งค่าเองผ่าน SQL แบบชัดเจนกว่า) ส่วน **"Enable Data API"** เปิดไว้ตามเดิม

### 2. สร้างตารางเก็บข้อมูล RSVP
SQL Editor → New query → วางทั้งหมดนี้แล้ว Run:

```sql
create table rsvp_responses (
  id uuid primary key default gen_random_uuid(),
  device_id text not null,
  name text not null,
  attend text not null,
  guests int not null default 1,
  message text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

alter table rsvp_responses enable row level security;

grant insert on rsvp_responses to public;
grant select on rsvp_responses to authenticated;

create policy "anyone can insert" on rsvp_responses
  for insert to public
  with check (true);

create policy "authenticated can read all" on rsvp_responses
  for select to authenticated
  using (true);
```

**แขกทั่วไป (ไม่ล็อกอิน) ส่งคำตอบได้ แต่อ่านคำตอบของคนอื่นไม่ได้เลย** มีแค่คุณ (ที่ล็อกอินแล้ว) เท่านั้นที่ดูรายชื่อทั้งหมดได้ผ่านหน้า Dashboard — สังเกตว่า `device_id` **ไม่ใส่ `unique`** เพราะสถาปัตยกรรมนี้ให้ทุกครั้งที่ส่ง/แก้ไข RSVP เป็นการเพิ่มแถวใหม่เสมอ (ดูหัวข้อถัดไป)

### 3. สร้างบัญชีล็อกอินสำหรับหน้า Dashboard
**Authentication** → **Users** → **Add user** → **Create new user** → ใส่อีเมล/รหัสผ่าน → ติ๊ก **Auto Confirm User** → Create

### 4. หา Project URL และคีย์
**Project Settings** → **API Keys** → แท็บ **"Publishable and secret API keys"** (ไม่ใช่แท็บ Legacy) → copy **Project URL** และ **publishable key** (`sb_publishable_...`)

### 5. ส่งค่ามาให้ผมใส่ในโค้ด
```
Project URL: https://xxxxxxxxxxxx.supabase.co
publishable key: sb_publishable_xxxxxxxxxxxxxxxxxxxxxxxx
```

## กลไก "แก้ไขคำตอบเดิมได้" ทำงานยังไง

ตอนแรกออกแบบให้ใช้ UPDATE (upsert) แก้ไขแถวเดิมตรงๆ แต่เจอข้อจำกัดของโปรเจกต์นี้ที่ทำให้ UPDATE ผ่าน API ใช้งานไม่ได้แม้ policy/สิทธิ์จะถูกต้องทุกอย่าง (สาเหตุยังไม่ชัดเจน 100% แต่ INSERT ใช้งานได้เสถียรมาก) จึงเปลี่ยนสถาปัตยกรรมเป็น **append-only**:

- ทุกครั้งที่กด "ส่งคำตอบ" หรือ "อัปเดตคำตอบ" ฝั่งเว็บจะ **insert แถวใหม่เสมอ** พร้อมแท็ก `device_id` เดิม (เก็บไว้ใน localStorage ของเบราว์เซอร์แขกแต่ละคน)
- หน้า **Dashboard** (`dashboard.html`) ดึงข้อมูลทั้งหมดมา แล้ว**กรองให้เหลือแค่แถวล่าสุด (`updated_at` มากสุด) ของแต่ละ `device_id`** ก่อนแสดงผลและคำนวณสรุปยอด — คู่บ่าวสาวจึงเห็นแค่คำตอบล่าสุดของแขกแต่ละคน ไม่งงกับแถวซ้ำ
- ถ้าอยากดูประวัติการแก้ไขทั้งหมดของใครสักคน เปิดดูตรงๆ ได้ที่ **Table Editor** → `rsvp_responses` → กรองด้วย `device_id`

## หลังตั้งค่าเสร็จ
- แขกกรอก RSVP ที่หน้าเว็บหลักตามปกติ ข้อมูลจะเข้า Supabase (ดูได้ใน **Table Editor** → `rsvp_responses` ก็ได้ ถ้าไม่อยากรอหน้า Dashboard)
- เข้าหน้าดูผลสวยๆ ได้ที่ `<โดเมนเว็บของคุณ>/dashboard.html` ล็อกอินด้วยอีเมล/รหัสผ่านจากข้อ 3
- มีแถวทดสอบค้างอยู่จากตอน debug (`simpletest001`, `minimal-check-001`, `e2e-final-check-001`, `final-append-check-001`) — ลบทิ้งได้ทั้งหมดผ่าน **Table Editor** (เลือกทุกแถว → Delete) ก่อนเริ่มใช้งานจริงกับแขก
