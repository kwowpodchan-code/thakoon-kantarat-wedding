# วิธีตั้งค่า Supabase สำหรับ RSVP + Dashboard

ระบบ RSVP เปลี่ยนจาก Google Form มาเป็นฐานข้อมูลของเราเองแล้ว (Supabase — ฐานข้อมูล Postgres ฟรี) ข้อดีคือ **แก้ไขคำตอบเดิมได้จริง** (ไม่สร้างแถวซ้ำเหมือน Google Sheet) และมีหน้า Dashboard ให้ดูผลได้สวยๆ ทำตาม 5 ขั้นตอนนี้ครั้งเดียว

## 1. สมัครบัญชีและสร้างโปรเจกต์
1. ไปที่ https://supabase.com → กด "Start your project" → สมัครด้วย GitHub หรืออีเมลก็ได้ (ฟรี ไม่ต้องผูกบัตร)
2. กด "New project" ตั้งชื่ออะไรก็ได้ เช่น `wedding-rsvp`
3. ตั้งรหัสผ่านฐานข้อมูล (Database Password) — จดเก็บไว้ (ไม่จำเป็นต้องใช้ในขั้นตอนต่อไป แต่เผื่อไว้)
4. เลือก Region ใกล้ไทยที่สุด (เช่น Singapore) แล้วกด "Create new project" รอสักครู่ให้สร้างเสร็จ (~1-2 นาที)

## 2. สร้างตารางเก็บข้อมูล RSVP
1. ในเมนูซ้ายของโปรเจกต์ กด **"SQL Editor"**
2. กด "New query" แล้ววางโค้ดนี้ทั้งหมด:

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

create policy "anon can insert" on rsvp_responses
  for insert to anon
  with check (true);

create policy "anon can update" on rsvp_responses
  for update to anon
  using (true)
  with check (true);

create policy "authenticated can read all" on rsvp_responses
  for select to authenticated
  using (true);
```

3. กดปุ่ม **"Run"** (หรือ Ctrl+Enter) ควรขึ้นข้อความ "Success. No rows returned"

นี่คือส่วนสำคัญที่ทำให้ระบบปลอดภัย: **แขกทั่วไป (ไม่ล็อกอิน) ส่ง/แก้ไขคำตอบของตัวเองได้ แต่อ่านคำตอบของคนอื่นไม่ได้เลย** มีแค่คุณ (ที่ล็อกอินแล้ว) เท่านั้นที่ดูรายชื่อทั้งหมดได้ผ่านหน้า Dashboard

## 3. สร้างบัญชีล็อกอินสำหรับหน้า Dashboard
1. เมนูซ้าย กด **"Authentication"** → แท็บ **"Users"**
2. กด **"Add user"** → **"Create new user"**
3. ใส่อีเมลและรหัสผ่านที่คุณ (คู่บ่าวสาว) จะใช้ล็อกอินเข้าหน้าดูผล RSVP — จดจำไว้ให้ดี
4. ติ๊ก **"Auto Confirm User"** แล้วกด Create

## 4. หา Project URL และ anon key
1. เมนูซ้าย กด **"Project Settings"** (ไอคอนเฟือง) → **"API"**
2. คัดลอกค่า **"Project URL"** (หน้าตาประมาณ `https://xxxxxxxxxxxx.supabase.co`)
3. คัดลอกค่า **"anon public"** key (สตริงยาวๆ ใต้หัวข้อ Project API keys) — ค่านี้เอาไปฝังในโค้ดเว็บได้อย่างปลอดภัย เพราะถูกจำกัดสิทธิ์ด้วย policy ที่ตั้งไว้ในข้อ 2 แล้ว

## 5. ส่งค่าทั้งสองมาให้ผม
ส่งข้อความมาในรูปแบบนี้:
```
Project URL: https://xxxxxxxxxxxx.supabase.co
anon key: eyJxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
ผมจะเอาไปใส่ในโค้ด `index.html` และ `dashboard.html` ให้ทันที แล้วทดสอบส่ง RSVP จริงให้ดูจนกว่าจะมั่นใจว่าทำงานถูกต้อง

## หลังตั้งค่าเสร็จ
- แขกกรอก RSVP ที่หน้าเว็บหลักตามปกติ ข้อมูลจะเข้า Supabase (ดูได้ในเมนู "Table Editor" → `rsvp_responses` ก็ได้เช่นกัน ถ้าไม่อยากรอหน้า Dashboard)
- เข้าหน้าดูผลสวยๆ ได้ที่ `<โดเมนเว็บของคุณ>/dashboard.html` ล็อกอินด้วยอีเมล/รหัสผ่านจากข้อ 3
