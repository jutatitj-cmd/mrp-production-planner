# ระบบวางแผนการผลิต (MRP) — ออเดอร์

เว็บแอปวางแผนการผลิตแบบไฟล์เดียว (static) ทำงานฝั่งเบราว์เซอร์ เหมาะกับ **GitHub Pages**
รองรับเก็บข้อมูลบน **Supabase** (sync หลายเครื่อง/หลายคน) หรือใช้ localStorage ในเครื่องก็ได้

โยนออเดอร์เข้าไป แล้วระบบคำนวณให้:

- 🗓️ **ส่งได้วันไหน** ต่อออเดอร์ (สถานะ ทัน/ช้ากี่วัน)
- 🧪 **วัตถุดิบขาดเท่าไร** (ต้องใช้ / มีอยู่ / ขาด) จากสูตร BOM
- 📦 **ของมีเท่าไร / ต้องผลิตเท่าไร** ต่อสินค้า
- 🚀 **เร่งส่งต้องเพิ่มกี่คน** เพื่อให้ทันกำหนด
- 📅 **แผนผลิตรายวัน** แต่ละวันต้องผลิตอะไรเท่าไร

## โมเดลการคำนวณ
- กำลังผลิต/วัน = `ชิ้นต่อรอบ × รอบต่อคนต่อวัน × จำนวนคน` (ตามรอบ/Cycle)
- ต้องผลิต (net) = `ยอดสั่งรวม − สต็อกที่มี`
- ผลิตวันละเท่ากำลังผลิต จนครบ → ได้วันเสร็จ/วันส่ง
- เพิ่มคน = เพิ่มกำลังผลิต → ใช้คำนวณ "เร่งส่ง"

## ตั้งค่า Supabase (ทางเลือก แต่แนะนำ)
1. เปิด Supabase project → เมนู **SQL Editor** → รัน SQL นี้ 1 ครั้ง:
   ```sql
   create table if not exists mrp_state (
     id text primary key,
     data jsonb not null default '{}'::jsonb,
     updated_at timestamptz default now()
   );
   alter table mrp_state enable row level security;
   create policy "mrp anon read"   on mrp_state for select using (true);
   create policy "mrp anon write"  on mrp_state for insert with check (true);
   create policy "mrp anon update" on mrp_state for update using (true) with check (true);
   ```
2. ไปที่ **Project Settings → API** คัดลอก **Project URL** และ **anon public key**
3. เปิดแอป → แท็บ **ตั้งค่า & ข้อมูล** → วาง URL + anon key → กด **เชื่อมต่อ & ซิงก์**

> ⚠️ ใช้ **anon key เท่านั้น** (ไม่ใช่ service_role) · key เก็บในเบราว์เซอร์ ไม่ถูก commit ขึ้น repo
> นโยบาย RLS ด้านบนเปิดให้ทุกคนที่มี URL+key อ่าน/เขียนได้ — ถ้าต้องการจำกัดสิทธิ์ เพิ่ม Supabase Auth ภายหลังได้

## Deploy ขึ้น GitHub Pages
1. อัปโหลด `index.html` ไว้ที่ root ของ repo (ผ่านหน้าเว็บ GitHub หรือ git push)
2. repo → **Settings → Pages → Source: Deploy from a branch** → เลือก `main` / `(root)` → Save
3. เว็บจะขึ้นที่ `https://<username>.github.io/<repo>/`

## การใช้งาน
1. แท็บ **สินค้า & กำลังผลิต** — ตั้งค่าสินค้า สต็อก และกำลังผลิต
2. แท็บ **วัตถุดิบ & BOM** — คลังวัตถุดิบ + สูตรว่าสินค้า 1 ชิ้นใช้อะไรเท่าไร
3. แท็บ **ตั้งค่า** — วันเริ่มผลิต, วันทำงาน, เชื่อม Supabase
4. แท็บ **ออเดอร์** — โยนออเดอร์เข้าไป
5. แท็บ **ผลการวางแผน** — ดูผลลัพธ์ทั้งหมด

> กดปุ่ม **โหลดข้อมูลตัวอย่าง** เพื่อทดลองใช้ทันที
