## Prompt

### Prompt 1.1
```
Context:
D:\Works\OICIIQE\Brain_IIQE\oic_iiqe_database_schema.md อันนี้คือไฟล์ Database Schema
D:\Works\OICIIQE\Brain_IIQE\oic_iiqe_database_ai_schema.csv อันนี้คือไฟล์ Database AI Schema

Problem:
จากหน้าจอ /ReportExaminationRate/Search
จำนวนผู้สมัครสอบ นายหน้าประกันชีวิต ปี 2568 ของ IIQE กับ Exam ตัวเลขไม่ตรงกันครับ
ระบบ exam จำนวน 63,956 คน
ระบบ IIQE จำนวน 63,180 คน
ต่างกัน 776 คนครับ
ในระบบ exam อาจจะมีรอบที่ย้ายไปแล้วค้างอยู่บ้าง แต่ไม่น่ามีจำนวนแตกต่างกับ IIQE มากขนาดนี้ครับ
```

### Prompt 1.2
```
Context:
Database: Oracle

Schema files:
- oic_iiqe_database_schema.md
- oic_iiqe_database_ai_schema.csv

Problem:
จากหน้าจอ /ReportExaminationRate/Search

จำนวนผู้สมัครสอบ "นายหน้าประกันชีวิต" ปี 2568
ตัวเลขจาก 2 ระบบไม่ตรงกัน

Exam System:
63,956 คน

IIQE System:
63,180 คน

Difference:
776 คน

Observation:
ในระบบ Exam อาจมีรอบที่ย้ายแล้ว (transfer) แต่ค้างอยู่บ้าง
แต่ไม่น่าทำให้ตัวเลขต่างกันถึง 776

Task:
1. วิเคราะห์ว่าความแตกต่างของตัวเลขอาจเกิดจากอะไร
2. แนะนำ SQL ที่ใช้ตรวจสอบ record ที่หายไป
3. แนะนำ SQL ที่ใช้เปรียบเทียบข้อมูลระหว่างสองระบบ
4. แนะนำจุดที่ควรตรวจสอบ เช่น
   - duplicate join
   - cancelled round
   - transfer round
   - absent vs attend
```