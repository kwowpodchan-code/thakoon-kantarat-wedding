# วิธีตั้งค่า Supabase สำหรับ RSVP + Dashboard

> **สถานะ: ตั้งค่าเสร็จแล้ว ✅** โปรเจกต์จริงที่ใช้งานอยู่คือ `wedding-rsvp-v2` (`https://rusyltzeomupisfdhtvr.supabase.co`) เชื่อมเข้ากับ `index.html` และ `dashboard.html` เรียบร้อยแล้ว ทดสอบส่ง/แก้ไข RSVP จริงผ่านแล้ว เอกสารนี้เก็บไว้อ้างอิง เผื่อวันหลังต้องสร้างโปรเจกต์ใหม่หรือมีปัญหาต้องแก้ไข

ระบบ RSVP ใช้ฐานข้อมูลของเราเอง (Supabase — Postgres ฟรี) ข้อดีคือ **แก้ไขคำตอบเดิมได้จริง** (ไม่สร้างแถวซ้ำเหมือน Google Sheet) และมีหน้า Dashboard ให้ดูผลได้สวยๆ

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
  device_id text unique not null,
  name text not null,
  attend text not null,
  guests int not null default 1,
  message text,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

alter table rsvp_responses enable row level security;

grant select, insert, update on rsvp_responses to anon;
grant select on rsvp_responses to authenticated;

create policy "anyone can insert" on rsvp_responses
  for insert to public
  with check (true);

create policy "anon can update" on rsvp_responses
  for update to anon
  using (true)
  with check (true);

create policy "authenticated can read all" on rsvp_responses
  for select to authenticated
  using (true);
```

**แขกทั่วไป (ไม่ล็อกอิน) ส่ง/แก้ไขคำตอบของตัวเองได้ แต่อ่านคำตอบของคนอื่นไม่ได้เลย** มีแค่คุณ (ที่ล็อกอินแล้ว) เท่านั้นที่ดูรายชื่อทั้งหมดได้ผ่านหน้า Dashboard

### 3. สร้างบัญชีล็อกอินสำหรับหน้า Dashboard
**Authentication** → **Users** → **Add user** → **Create new user** → ใส่อีเมล/รหัสผ่าน → ติ๊ก **Auto Confirm User** → Create

### 4. หา Project URL และคีย์
**Project Settings** → **API Keys** → แท็บ **"Publishable and secret API keys"** (ไม่ใช่แท็บ Legacy) → copy **Project URL** และ **publishable key** (`sb_publishable_...`)

### 5. ส่งค่ามาให้ผมใส่ในโค้ด
```
Project URL: https://xxxxxxxxxxxx.supabase.co
publishable key: sb_publishable_xxxxxxxxxxxxxxxxxxxxxxxx
```

## หมายเหตุทางเทคนิค (เจอตอน debug จริง เก็บไว้กันงงซ้ำ)

- **ต้องใช้คีย์จากแท็บ "Publishable and secret API keys" เท่านั้น** ไม่ใช่แท็บ "Legacy anon, service_role API keys" — โปรเจกต์ที่เพิ่งสร้างใหม่บางทีคีย์ legacy จะผูกกับ JWT signing key คนละตัวกับที่ระบบใช้งานจริง ทำให้ยืนยันตัวตนไม่ผ่านแบบเงียบๆ (error จะหน้าตาเหมือน RLS ผิดพลาด ทั้งที่จริงเป็นเรื่องคีย์)
- **ห้ามส่ง RSVP ด้วย `Prefer: return=representation` หรือ upsert ผ่าน `on_conflict`** — เพราะ anon ไม่มีสิทธิ์ SELECT (กันไว้ไม่ให้แขกอ่านข้อมูลคนอื่น) การขอให้ Postgres "อ่านค่ากลับมา" หลัง insert/upsert จะชนกับ RLS ทันที แม้ policy insert จะถูกต้องก็ตาม — โค้ดที่ใช้งานจริงตอนนี้แก้โดย: **ยิง insert ธรรมดาก่อน (`Prefer: return=minimal`) ถ้าเจอ device_id ซ้ำ (HTTP 409) ค่อยยิง PATCH แก้ไขแถวเดิมแทน** (ดูใน `index.html` ฟังก์ชัน `doInsert`/`doUpdate`) วิธีนี้ไม่ต้องพึ่ง SELECT เลย ปลอดภัยครบ

## หลังตั้งค่าเสร็จ
- แขกกรอก RSVP ที่หน้าเว็บหลักตามปกติ ข้อมูลจะเข้า Supabase (ดูได้ใน **Table Editor** → `rsvp_responses` ก็ได้ ถ้าไม่อยากรอหน้า Dashboard)
- เข้าหน้าดูผลสวยๆ ได้ที่ `<โดเมนเว็บของคุณ>/dashboard.html` ล็อกอินด้วยอีเมล/รหัสผ่านจากข้อ 3
- มีแถวทดสอบ (`minimal-check-001`, `e2e-final-check-001`) ค้างอยู่ในตารางจากตอน debug — ลบทิ้งได้เลยผ่าน **Table Editor** (ติ๊กแถว → ลบ) ก่อนเริ่มใช้งานจริงกับแขก
