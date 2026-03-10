# Project Proposal

## Project Name
**Nexus Academy AI — The Task-Life Harmonizer**

---

## Problem Statement

**WHO ใครเดือดร้อน(?)**
- นักศึกษามหาวิทยาลัยที่ต้องจัดการงานหลายวิชาพร้อมกัน
- วัยทำงานที่มีหลายโปรเจ็คในเวลาเดียวกัน
- บุคคลทั่วไปที่มีตารางชีวิตแน่นและใช้ LINE เป็นเครื่องมือสื่อสารหลัก

**WHAT ปัญหาคืออะไร(?)**
- ข้อมูลงานและ deadline กระจายอยู่หลายที่ เช่น LINE chat, อีเมล และคำสั่งในห้องเรียน
- ผู้ใช้ต้องจำเองหรือจดหลายที่ ทำให้เกิดความสับสนและมีโอกาสลืมงาน

**WHEN เกิดบ่อยแค่ไหน(?)**
- เกิดขึ้นบ่อยโดยเฉพาะช่วง Midterm, Final หรือช่วงที่มีหลายโปรเจ็คต้องส่งพร้อมกัน

**HOW MUCH เสียเวลาเงิน(/ เท่าไหร่?)**
- ผู้ใช้ต้องเสียเวลาเฉลี่ยประมาณ 15–30 นาทีต่อวันในการเช็คข้อมูลจากหลายแหล่ง
- มีความเสี่ยงที่จะส่งงานล่าช้าหรือพลาด deadline

---

## Proposed Solution

สร้างระบบ **Nexus Academy AI** ซึ่งเป็นผู้ช่วยอัจฉริยะที่ช่วยจดงาน วางแผน และแจ้งเตือน deadline ผ่าน LINE

ผู้ใช้สามารถพิมพ์ข้อความธรรมดา เช่น กำหนดส่งงาน Proposal CSI403 ภายในวันที่ 13 มี.ค. พ.ศ.2569


ระบบจะใช้ AI วิเคราะห์ข้อความและสกัดข้อมูลสำคัญ เช่น

- ชื่องาน
- วิชา
- วันส่ง
- ประเภทงาน

จากนั้นข้อมูลจะถูกบันทึกลงฐานข้อมูล และระบบจะส่งการแจ้งเตือนอัตโนมัติเมื่อใกล้ deadline

ระบบนี้ช่วยรวมข้อมูลทุกอย่างไว้ในที่เดียว ลดภาระการจำงาน และใช้แพลตฟอร์ม LINE ที่ผู้ใช้คุ้นเคยอยู่แล้ว

---

## n8n Workflow Concept

**Trigger**
- Webhook จาก LINE Messaging API เมื่อผู้ใช้ส่งข้อความเข้ามา

**Process**
- ใช้ Gemini AI วิเคราะห์ข้อความเพื่อสกัดข้อมูล
- ใช้ Code Node แปลงผลลัพธ์เป็น JSON
- ใช้ IF Node ตรวจสอบประเภทคำสั่ง เช่น เพิ่มงาน หรือ mark งานเสร็จ
- ใช้ Schedule Trigger ตรวจสอบ deadline เป็นระยะ

**Action**
- บันทึกข้อมูลลง Google Sheets
- อัปเดตสถานะงานเมื่อผู้ใช้แจ้งว่างานเสร็จแล้ว

**Notify**
- ส่งข้อความตอบกลับผ่าน LINE Messaging API
- ส่ง Morning Brief ทุกเช้า
- แจ้งเตือนเมื่อ deadline ใกล้เข้ามา

---

## APIs & Tools

- [ ] LINE Messaging API  
https://developers.line.biz/en/docs/messaging-api/

- [ ] Gemini API (Google AI Studio)  
https://ai.google.dev/docs

- [ ] n8n Workflow Automation  
https://docs.n8n.io/

- [ ] Google Sheets API  
https://developers.google.com/sheets/api

---

## GitHub Repository
https://github.com/RelaxTimee/CSI403_Nexus.git

---

## Expected Outcome

- ลดการลืม deadline ลงอย่างมีนัยสำคัญ
- ลดเวลาการจัดการตารางงานลงประมาณ 15–30 นาทีต่อวัน
