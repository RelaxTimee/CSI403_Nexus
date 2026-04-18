# Nexus AI — The Task-Life Harmonizer

บอทจัดการงานและ deadline อัตโนมัติผ่านแอปพลิเคชัน LINE พัฒนาด้วย n8n เพื่อช่วยให้นักศึกษาและวัยทำงานไม่พลาด deadline โดยไม่ต้องเฝ้าจำหรือจดไว้หลายที่

---

## Problem & Solution

- **ปัญหา:** ข้อมูลงานและ deadline กระจายอยู่หลายที่ทั้ง LINE chat, อีเมล และคำสั่งในห้องเรียน ทำให้ผู้ใช้เสียเวลาในการตรวจสอบ และมีความเสี่ยงส่งงานล่าช้าหรือพลาด deadline
- **ทางแก้:** ระบบ AI ที่รับข้อความภาษาธรรมชาติผ่าน LINE วิเคราะห์ด้วย LLM สกัดข้อมูลงาน บันทึกลง Google Sheets และแจ้งเตือนอัตโนมัติเมื่อ deadline ใกล้เข้ามา รองรับการสนทนาหลายรอบ (multi-turn) และตรวจจับนัดซ้อน (conflict detection)

---

## System Architecture

ระบบถูกออกแบบเป็น 3 Workflow บน n8n ดังนี้

### Workflow 1 — Main Chat Flow (รับ-ตอบ LINE)

```mermaid
flowchart TD
    A([Webhook Trigger\nPOST /webhook/nexus]) --> B
    B[Edit Fields\nดึง userId, replyToken, text] --> C
    C[GSheet Read Session\nอ่าน session ค้างของผู้ใช้] --> D
    D[Code Merge Session\nรวม session เดิม + ข้อความใหม่] --> E
    E[HTTP Request\nLLM API — วิเคราะห์ข้อความ] --> F
    F["{ } Code Parse LLM\nparse JSON + merge session data"] --> G
    G{Switch Intent\nadd / complete / view\ndelete / edit / unknown}

    G -->|add_task incomplete| H1[GSheet Save Session\nบันทึก session รอข้อมูลเพิ่ม]
    G -->|add_task complete| H2[GSheet Check Conflict\nตรวจนัดซ้อนวันเดียวกัน]
    G -->|complete_task| H3[GSheet Find Task\nค้นหางานที่จะ done]
    G -->|view_tasks| H4[GSheet Get Tasks\nดึงงาน pending ทั้งหมด]
    G -->|delete_task| H5[GSheet Find Task\nค้นหางานที่จะลบ]
    G -->|edit_task| H6[GSheet Find Task\nค้นหางานที่จะแก้ไข]
    G -->|unknown| H7[Code Unknown Intent\nสร้างข้อความแนะนำ]

    H1 --> R
    H2 --> I2[Code Detect Conflict\nเช็คเวลาทับซ้อน ± 60 นาที]
    I2 --> J2{Switch Conflict}
    J2 -->|no conflict| K2[GSheet Add Task\nบันทึกงานใหม่]
    J2 -->|has conflict| R
    K2 --> L2[GSheet Clear Session\nลบ session row]
    L2 --> R

    H3 --> I3[Code Prep Complete\nเตรียม row_number]
    I3 --> J3[GSheet Update Status Done\nอัปเดต status = done]
    J3 --> R

    H4 --> I4["{ } Code Format Task List\nจัดเรียง + urgency indicator"]
    I4 --> R

    H5 --> I5[Code Prep Delete\nเตรียม row_number]
    I5 --> J5{Switch Skip Delete\nตรวจ skip_delete flag}
    J5 -->|found| K5[GSheet Delete Task\nอัปเดต status = deleted]
    K5 --> R
    J5 -->|not found| R

    H6 --> I6[Code Prep Edit\nเตรียมข้อมูลใหม่]
    I6 --> J6[GSheet Update Task Edit\nอัปเดต deadline ใหม่ + reset flags]
    J6 --> R
    H7 --> R

    R[Code Merge Reply\nรวม replyToken + message] --> S
    S[LINE Reply API\nตอบกลับผู้ใช้] --> Z([สิ้นสุด])
```

---

### Workflow 2 — Deadline Checker (แจ้งเตือนทุก 15 นาที)

```mermaid
flowchart TD
    A1([Schedule Trigger\nทุก 15 นาที]) --> B1
    B1[GSheet Get Pending Tasks\nดึงงาน status = pending ทั้งหมด] --> C1
    C1["{ } Code Check Deadline Hours\nคำนวณเวลาที่เหลือ (Asia/Bangkok)"] --> D1
    D1{มีงานที่ต้องแจ้งเตือน?}
    D1 -->|notified_15m = false\nและ ≤ 15 นาที| E1[LINE Push API\nแจ้ง อีก 15 นาที]
    D1 -->|notified_1h = false\nและ ≤ 60 นาที| E1
    D1 -->|ไม่มี| E2[No Operation\nข้ามไป]
    E1 --> F1["{ } Code Prep Notified Flag\nเตรียม flag ที่ถูกต้อง\n(ไม่ reset flag เดิม)"]
    F1 --> G1[GSheet Update Notified Flag\nอัปเดต notified_1h / notified_15m]
```

---

### Workflow 3 — Morning Brief (ทุกวัน 07:00 น.)

```mermaid
flowchart TD
    A2([Schedule Trigger\nทุกวัน 07:00 น.]) --> B2
    B2[GSheet Get All Pending\nดึงงาน pending ทั้งหมดทุก user] --> C2
    C2["{ } Code Build Morning Brief\nจัดกลุ่มตาม user_id\nแบ่งเป็น overdue / ใน 7 วัน / ทั้งหมด"] --> D2
    D2[LINE Push Morning Brief\nส่งสรุปงานประจำวันให้แต่ละ user]
```

---

## ตัวอย่างการใช้งาน

| ข้อความที่พิมพ์ | ผลลัพธ์ |
|---|---|
| `กำหนดส่งงาน Proposal CSI403 ภายในวันที่ 13 มี.ค.` | บันทึกงาน + ยืนยันกลับ |
| `มีแข่ง 10 โมงพรุ่งนี้` | บันทึกงานประเภท Meeting พร้อมแจ้งถ้านัดซ้อน |
| `ทำ Report CSI401 เสร็จแล้ว` | อัปเดตสถานะเป็น done |
| `มีงานอะไรบ้าง` | แสดงรายการงาน pending ทั้งหมด พร้อม urgency |
| `เลื่อนงาน Proposal เป็นวันที่ 20` | แก้ไข deadline + reset การแจ้งเตือน |
| `ลบงาน Proposal` | ลบงานออกจากรายการ |

---

## การแจ้งเตือน

| ประเภท | เวลา | ช่องทาง |
|---|---|---|
| Morning Brief | ทุกวัน 07:00 น. | LINE Push |
| แจ้งเตือนก่อน 1 ชั่วโมง | ก่อน Deadline 1 ชั่วโมง | LINE Push |
| แจ้งเตือนก่อน 15 นาที | ก่อน Deadline 15 นาที | LINE Push |

> งานที่ไม่มีเวลา (`has_time = false`) จะใช้เวลา 07:00 น. เป็น default สำหรับการคำนวณ deadline

---
