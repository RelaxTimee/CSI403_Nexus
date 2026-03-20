# Nexus AI — The Task-Life Harmonizer
 
บอทจัดการงานและ deadline อัตโนมัติผ่านแอปพลิเคชัน LINE พัฒนาด้วย n8n เพื่อช่วยให้นักศึกษาและวัยทำงานไม่พลาด deadline โดยไม่ต้องเฝ้าจำหรือจดไว้หลายที่
 
---
 
## Problem & Solution
 
- **ปัญหา:** ข้อมูลงานและ deadline กระจายอยู่หลายที่ทั้ง LINE chat, อีเมล และคำสั่งในห้องเรียน ทำให้ผู้ใช้เสียเวลาเฉลี่ย 15–30 นาทีต่อวันในการตรวจสอบ และมีความเสี่ยงส่งงานล่าช้าหรือพลาด deadline
- **ทางแก้:** ระบบ AI ที่รับข้อความภาษาธรรมชาติผ่าน LINE วิเคราะห์ด้วย Gemini AI สกัดข้อมูลงาน บันทึกลง Google Sheets และแจ้งเตือนอัตโนมัติเมื่อ deadline ใกล้เข้ามา
 
---
 
## System Architecture 
 
ระบบถูกออกแบบเป็น 2 Workflow บน n8n ดังนี้
 
### Workflow 1 — Main (รับ-ตอบ LINE)
 
```mermaid
flowchart TD
    A([ Webhook Trigger\nPOST /webhook/nexus]) --> B
    B[ Edit Fields\nดึง userId, replyToken, text] --> C
    C[ HTTP Request\nGemini API — วิเคราะห์ข้อความ] --> D
    D["{ } Code in JavaScript\nparse JSON จาก Gemini"] --> E
    E{ Switch\nadd / complete / view / unknown}
 
    E -->|add_task| F1[ Google Sheets\nAppend Row]
    E -->|complete_task| F2[ Google Sheets\nUpdate Row]
    E -->|view_tasks| F3[ Google Sheets\nGet Rows]
    E -->|unknown| F4[ Edit Fields\nข้อความแจ้ง error]
 
    F3 --> G["{ } Code in JavaScript\nformat รายการงาน"]
 
    F1 --> H
    F2 --> H
    G --> H
    F4 --> H
 
    H[ HTTP Request\nLINE Reply API — ตอบกลับผู้ใช้] --> Z([สิ้นสุด])
```
 
---
 
### Workflow 2 — Scheduler (แจ้งเตือน deadline + Morning Brief)
 
```mermaid
flowchart TD
    subgraph LEFT["Deadline Checker (ทุก 30 นาที)"]
        A1([Schedule Trigger\nทุก 30 นาที]) --> B1
        B1[Google Sheets\nดึงงาน status = pending] --> C1
        C1["{ } Code in JavaScript\nคำนวณ hours to deadline"] --> D1
        D1{IF Node\ndeadline ≤ 24h\nและยังไม่แจ้ง?}
        D1 -->|true| E1[HTTP Request\nLINE Push + update flag]
        D1 -->|false| E2[No Operation\nข้ามไป]
    end
 
    subgraph RIGHT["Morning Brief (ทุกวัน 07:00 น.)"]
        A2([Schedule Trigger\nทุกวัน 07:00 น.]) --> B2
        B2[Google Sheets\nดึงงานทั้งหมดวันนี้] --> C2
        C2["{ } Code in JavaScript\nจัดเรียงตาม deadline"] --> D2
        D2[HTTP Request\nGemini — สร้าง Morning Brief] --> E3
        E3[HTTP Request\nLINE Push API — ส่ง Morning Brief]
    end
```
 
---
 
## ตัวอย่างการใช้งาน
 
| ข้อความที่พิมพ์ | ผลลัพธ์ |
|---|---|
| `กำหนดส่งงาน Proposal CSI403 ภายในวันที่ 13 มี.ค.` | บันทึกงาน + ยืนยันกลับ |
| `ทำ Report CSI401 เสร็จแล้ว` | อัปเดตสถานะเป็น done |
| `มีงานอะไรบ้างวันนี้` | แสดงรายการงานวันนี้ทั้งหมด |
| `งานที่ใกล้ deadline` | แสดงงานที่ครบกำหนดภายใน 3 วัน |
 
---
