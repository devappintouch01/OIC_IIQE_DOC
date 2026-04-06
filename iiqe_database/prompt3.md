## Prompt 

## รอบสอบ 6830300106
### Prompt 3.1
```
Context:
Database: Oracle

Schema files:
- Brain_IIQE\iiqe_database\oic_iiqe_database_schema.md
- Brain_IIQE\iiqe_database\oic_iiqe_database_ai_schema.csv
- Brain_IIQE\iiqe_database\oic_iiqe_primary_key.csv
- Brain_IIQE\iiqe_database\oic_iiqe_relationship_candidates.csv
- Brain_IIQE\iiqe_database\6830300106.jpg

Observation:
รอบ 6830300106 ของ TII มีบางคนย้ายรอบไป ทำให้ลำดับที่กระโดด แต่คนที่ไม่ย้ายก็จ่ายค่าสมัครสอบแล้ว เลยถึอว่ารอบนี้ยังมีอยู่ด้วย แต่คนที่ไม่ย้ายถือว่าขาดสอบครับ
เป็นสาเหตุที่ทำให้ยอดผู้สมัครสอบ นายหน้าประกันชีวิต ปี 2568 ของ IIQE กับ Exam ตัวเลขไม่ตรงกันหรือเปล่า

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"
```

### Prompt 3.2
```
Context:
Database: Oracle

Schema files:
- Brain_IIQE\iiqe_database\oic_iiqe_database_schema.md
- Brain_IIQE\iiqe_database\oic_iiqe_database_ai_schema.csv
- Brain_IIQE\iiqe_database\oic_iiqe_primary_key.csv
- Brain_IIQE\iiqe_database\oic_iiqe_relationship_candidates.csv
- Brain_IIQE\iiqe_database\6830300106.jpg

Observation:
- สิ่งที่สรุปกับ Owner ไป
รอบสอบ 6830300106
วันที่จัดสอบ วันเสาร์ที่ 29 มีนาคม 2568 หลังเกิดแผ่นดินไหว 1 วัน
ประเภท TII นายหน้าประกันชีวิต
จำนวนผู้สมัครเดิม 45 คน (ระบบ IIQE) / 49 คน (ฝั่ง Excel)

ย้ายรอบออกไปก่อน 43 คน
ขาดสอบ (ไม่มีบันทึกผล) 2 คน
1 ศิรินภา อุ่นกาศ 1119901954531 ที่นั่ง 36
2 ธำรงค์ศักดิ์ สินธุรักษ์ 3190400303136 ที่นั่ง 15

จำนวนที่ต่างกัน 4 คน คือ เลขบัตร 1600100649524, 1841601147920, 1104700021397, 1959800140549

ตรวจสอบจากตาราง TEMP_EXAM_RESULT ไม่มี record ของรอบ 6830300106 ครับ

- Owner ขอมาครับ
จากรอบสอบ 6830300106 ช่วยเสนอวิธีที่จะแก้ไขให้ข้อมูลจำนวนผู้สมัครสอบ จำนวนผู้เข้าสอบ และผลสอบ ของเคสนี้ให้รายงานสถิติที่เรียกในระบบถูกต้องหน่อยนะครับ

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"
```