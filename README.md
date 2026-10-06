# ROV Remote (ไม่ใช้ Blynk)

เว็บรีโมตคุม ESP32 ROV ผ่าน **GitHub Pages + Firebase (Google)**
- Login ด้วย Google • เจ้าของ = Gmail เดียวที่กำหนด • ผู้ควบคุมได้ทีละ **1 คน**
- คนนอกต้อง: ส่งคำขอ → เจ้าของกดอนุมัติ → ระบบสุ่มรหัส 15 ตัว (ไทย 3 / อังกฤษ 4 / เลข 5 / พิเศษ 3) → เจ้าของบอกรหัส → ผู้ขอพิมพ์ให้ถูก (ใช้ได้ครั้งเดียว หมดอายุ 5 นาที)
- การตรวจรหัสและสิทธิ์ทำที่ **Security Rules ฝั่งเซิร์ฟเวอร์** (`database.rules.json`) ไม่ใช่ในเบราว์เซอร์ จึงแก้จากหน้าเว็บไม่ได้

## ไฟล์
`index.html` (เว็บทั้งหมด) • `database.rules.json` (กฎความปลอดภัย) • `README.md`

## ติดตั้ง
1. console.firebase.google.com → สร้างโปรเจกต์ (แพ็กเกจ Spark ฟรี ไม่ต้องใช้บัตร)
2. **Authentication** → Sign-in method → เปิด **Google**; แล้ว Settings → Authorized domains → เพิ่ม `ชื่อคุณ.github.io`
3. **Realtime Database** → Create → เลือก region `asia-southeast1`
4. Authentication → Users → Add user (email/password) สำหรับ **บอร์ด ESP32** → คัดลอก **User UID**
5. แก้ `database.rules.json`: แทน `OWNER_EMAIL@gmail.com` (ทุกจุด) และ `DEVICE_UID` ด้วยค่าจริง → วางในแท็บ Rules → Publish
6. Project settings → Your apps → Web → คัดลอก config ใส่ `CFG` ใน `index.html` พร้อม `owner` = Gmail เจ้าของ
7. อัปโหลด 3 ไฟล์ขึ้น GitHub → Settings → Pages → Deploy from branch (`main` / root)
8. (แนะนำ) เปิด **App Check** และจำกัด API key ให้ใช้ได้เฉพาะโดเมน github.io ของคุณ

## โครงข้อมูล (ใช้ V เดิมทั้ง 43 ตัว)
**เว็บ → บอร์ด** `cmd/Vn` (บอร์ดฟังด้วย stream; ปุ่มกด = 1 ตอนกด, 0 ตอนปล่อย)

| ชนิด | Pin |
|---|---|
| ปุ่มกดค้าง | V6 Emergency, V8/V9 เอียง ซ้าย/ขวา, V11/V12 ขึ้น/ลง |
| Push | V38 ถ่ายรูป, V39 Reset Emergency (ทำงานเมื่อ =1) |
| Switch | V1 Lock ท่า, V2 Lock ลึก, V29/V30 วาล์ว, V37 ARM, V40 Manual, V41 Fwd, V42 Rev |
| Joystick | V10 เลี้ยว, V26 หน้า/หลัง (-255..255) |
| Slider | V43 ESC 0..100 |
| ตั้งค่า | V34 (1–3600 s), V35 (0.5–2 bar), V36 (0.5–5 bar) |

**เจ้าของ → บอร์ด** `wifi/V24` (SSID), `wifi/V25` (รหัส) — เขียนได้เฉพาะเจ้าของ บอร์ดควรลบค่าทิ้งหลังใช้

**บอร์ด → เว็บ** `tele/Vn` + `tele/ts` (ServerValue.TIMESTAMP ทุกครั้งที่ส่ง ใช้เป็นสถานะออนไลน์)
V0 (URL กล้อง http/https), V3–V5, V7, V13–V15, V16–V23, V27, V28, V31–V33 (ข้อความ), V32 (ไฟเตือน), และค่าสะท้อนจริงของ V1, V2, V29, V30, V34–V37, V40–V43 (ปุ่มบนเว็บแสดง **ค่าที่บอร์ดรายงาน** ไม่ใช่ค่าที่กด)

**ระบบสิทธิ์** `requests/{uid}`, `grants/{uid}` (เจ้าของเท่านั้น), `controller` = {uid, hb, …} ผู้ควบคุมปัจจุบัน

## ข้อกำหนดฝั่งบอร์ด (ต้องทำในเฟิร์มแวร์)
1. ล็อกอินด้วยบัญชีบอร์ดแล้ว stream `cmd` และ `wifi`; ส่ง telemetry เฉพาะค่าที่เปลี่ยน (ใช้ตรรกะ `txNeed/txSend` เดิม) เพื่อไม่ให้เกินโควตา
2. **Watchdog:** เว็บเขียน `controller/hb` ทุก 2 วินาที ถ้า `controller` ว่างหรือ hb เก่ากว่า ~5 วินาที ให้ถือว่าลิงก์หลุด (zero joystick/ปุ่ม เหมือนกรณี Blynk หลุดเดิม)
3. เว็บส่งศูนย์ให้ทุกปุ่ม/จอยเมื่อปล่อย, สลับแท็บ, ปิดหน้า, หรือถูกปลด แต่บอร์ดต้องไม่พึ่งแค่ข้อนี้

## หมายเหตุความปลอดภัย
- ห้ามใส่ Database secret / service account ใน repo (config ในเว็บเป็นข้อมูลสาธารณะตามปกติของ Firebase)
- รหัสที่สุ่มเก็บใน `grants/{uid}` อ่านได้เฉพาะเจ้าของ และถูกลบเมื่อผู้ขอเข้าควบคุมสำเร็จ/ถูกปลด/ปฏิเสธ
- ผู้ควบคุมที่ไม่ส่ง heartbeat เกิน 15 วินาที ถูกแทนที่ได้ (ป้องกันช่องควบคุมค้าง)
- โควตา Spark: 100 การเชื่อมต่อพร้อมกัน, ดาวน์โหลด 10 GB/เดือน — ไม่มีเพดานจำนวนข้อความแบบ Blynk
- การควบคุมยานใต้น้ำผ่านอินเทอร์เน็ตมีความหน่วง ทดสอบในสระและทดสอบ Emergency/Watchdog ก่อนใช้งานจริงเสมอ

## ขั้นต่อไป
แก้เฟิร์มแวร์ `ESP32_Main_ROVregen26.ino`: ถอด Blynk ออกถาวร แล้วแทน `BLYNK_WRITE(Vn)` / `Blynk.virtualWrite` ด้วย Firebase ตามตารางข้างบน
