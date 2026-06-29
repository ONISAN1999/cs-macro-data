# 🌐 CS Macro Data — Remote Macros Repository

Admin แก้ไฟล์ `macros.json` ที่นี่ → CS Macro App ทุกเครื่อง sync ใหม่อัตโนมัติ

## 📋 ขั้นตอน Setup ครั้งแรก

### 1️⃣ สร้าง GitHub Repo

1. ไปที่ https://github.com/new
2. Repository name: `cs-macro-data`
3. Owner: **ONISAN1999** (ต้องตรงกับใน app — ถ้าใช้ user อื่นต้องแก้ `REMOTE_MACROS_URL` ในแอป)
4. Visibility: **Public** (ต้อง public เพื่อให้ raw URL เข้าถึงได้โดยไม่ต้อง auth)
5. Initialize: ✓ Add README
6. กด **Create repository**

### 2️⃣ Push ไฟล์ macros.json

**Option A: Web UI (ง่ายสุด)**

1. ไปที่ repo ใหม่
2. กด **Add file → Upload files**
3. ลากไฟล์ `macros.json` (ในโฟลเดอร์นี้) → upload
4. กด **Commit changes**

**Option B: Terminal**

```bash
cd cs-macro-data
git init
git remote add origin https://github.com/ONISAN1999/cs-macro-data.git
git add macros.json README.md
git commit -m "init: starter macros"
git branch -M main
git push -u origin main
```

### 3️⃣ ทดสอบ Raw URL

เปิด:
```
https://raw.githubusercontent.com/ONISAN1999/cs-macro-data/main/macros.json
```

ถ้าเห็น JSON = พร้อมใช้ ✅

---

## 🔄 วิธีอัปเดต Macro ในอนาคต (Workflow ปกติ)

### ขั้นตอน (ทำผ่าน Web UI ของ GitHub)

1. ไปที่ `macros.json` ใน repo
2. กด **✏️ ดินสอ** (Edit this file)
3. แก้ไข JSON ตามต้องการ:
   - **เพิ่ม Macro ใหม่** → push object เข้า `macros[]`
   - **แก้ Macro เดิม** → แก้ field ใน object ที่ id ตรงกัน
   - **ลบ Macro** → set `"deleted": true` (อย่าลบ object ออก — เพื่อให้ CS ที่มี macro นั้นถูกลบด้วย)
4. **เพิ่มค่า `version` ขึ้น 1** (สำคัญ! ถ้าไม่เพิ่ม CS จะไม่ sync)
5. อัปเดต `updated_at`
6. กด **Commit changes** ที่ล่างหน้า

CS จะได้รับ macro ใหม่:
- **ทันที** ถ้าเปิดแอปอยู่และกด "ตรวจสอบ" ใน About
- **ครั้งถัดไป** ที่เปิดแอป (auto-sync หลังเปิดแอป 3 วินาที)

---

## 📐 Schema (โครงสร้างไฟล์)

```json
{
  "version": 2,
  "updated_at": "2026-07-01",
  "macros": [
    {
      "id": 1001,
      "title": "🤝 ทักทายลูกค้า",
      "category": "chat-nv",
      "content": "สวัสดี[ครับ/ค่ะ] แอดมิน {ชื่อ_CS}",
      "favorite": false,
      "hotkey": "HELLO",
      "tags": "ทักทาย,greeting"
    }
  ],
  "categories": []
}
```

### 🔑 Field Reference

| Field | Required | คำอธิบาย |
|---|---|---|
| `id` | ✅ | number unique · **ใช้ 1000+** เพื่อไม่ทับ default (1-30) |
| `title` | ✅ | ชื่อบนการ์ด |
| `category` | ✅ | id หมวด → `chat-nv` หรือ `timer` |
| `content` | ✅ | เนื้อหา · `{ตัวแปร}` ให้กรอก · `[ครับ/ค่ะ]` เปลี่ยนตาม gender |
| `favorite` | optional | default `false` |
| `hotkey` | optional | คีย์ลัด aText (ASCII only · case-sensitive) |
| `tags` | optional | คำค้นหา คั่นด้วย comma |
| `deleted` | optional | set `true` เพื่อลบ macro ออกจากเครื่อง CS |

---

## 🛡 Merge Strategy (สำคัญ!)

เวลา CS sync จากที่นี่:

- ✅ **Macro ใหม่** (id ไม่มีในเครื่อง) → **เพิ่ม**
- ✅ **Macro เดิม** (id ตรงกัน) → **อัปเดต content/title/tags** แต่ **คง favorite state** ของ user
- ⚠️ **Macro local** (CS เคยสร้างเอง ก่อนล็อก) → **ไม่แตะ** (admin ลบไม่ได้)
- 🗑 **Macro ที่ admin set `deleted:true`** → **ลบออกจากเครื่อง CS**

---

## 📝 ตัวอย่าง: เพิ่ม Macro ใหม่

```json
{
  "id": 1005,
  "title": "💳 แจ้งเลขออเดอร์ที่จ่ายผ่าน QR",
  "category": "chat-nv",
  "hotkey": "QR",
  "tags": "QR,ชำระเงิน,order",
  "content": "ลูกค้าชำระเงินผ่าน QR Code สำเร็จ\nเลข Order: {Order_No}\nยอด: {ยอดเงิน} บาท\nขอบคุณ[ครับ/ค่ะ]"
}
```

แล้วเพิ่ม `version` จาก 1 → 2 → commit

---

## 🐛 Troubleshooting

**CS ไม่ได้รับ macro ใหม่:**
1. เช็คว่า `version` ใน macros.json มากกว่าใน app เครื่อง CS
2. เปิด CS Macro → About → กดปุ่ม "🔄 ตรวจสอบ" → ดู toast
3. ถ้าขึ้น "ตรวจสอบไม่สำเร็จ" → ตรวจ raw URL ว่าเข้าถึงได้

**Macro ลบไม่ออก:**
- ต้อง set `"deleted": true` (ไม่ลบ object) + เพิ่ม version

---

Built with 💚 by Fal_CXI · CS Macro v3.4.23+
