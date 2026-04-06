## Prompt 

## รอบสอบ 6810402513
### Prompt 4.1
```
Context:
Database: Oracle

Schema files:
- Brain_IIQE\iiqe_database\oic_iiqe_database_schema.md
- Brain_IIQE\iiqe_database\oic_iiqe_database_ai_schema.csv
- Brain_IIQE\iiqe_database\oic_iiqe_primary_key.csv
- Brain_IIQE\iiqe_database\oic_iiqe_relationship_candidates.csv
- Brain_IIQE\iiqe_database\6810402513.jpg

Observation:
จากในภาพ รอบ 6810402513 มีจำนวน 0 คน
อยากทราบว่าข้อมูลรอบสอบยนี้ ส่งเข้ามาที่ IIQE ได้อย่างไร
ถ้าส่งเข้ามา ที่ IIQE จะมีข้อมูลเป็นอย่างไร

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"
```

```
Context:
File bruno - D:\Works\OICIIQE\Brain_IIQE\OIC_IIQE_POSTMAN\Bruno\collections\1_DOPA_PROD_v3

Observation:
ต้อง migrate DOPA จาก v2 ไป v3
API Spec ของ DOPA v3 ตาม path bruno

Task:
1. ตรวจสอบ Code ปัจจุบันว่า ได้มีการแก้ไข DOPA จาก v2 ไป v3
ตามไฟล์ bruno แล้วหรือไม่
```

```
Context:
File bruno - D:\Works\OICIIQE\Brain_IIQE\OIC_IIQE_POSTMAN\Bruno\collections\*

Observation:
จากไฟล์ RegisterGeneral.cshtml เป็นหน้าสมัครสมาชิก IIQE
มี field รับข้อมูล lblFirstNameTH, lblLastNameTH เมื่อผู้ใช้กรอกข้อมูลแล้ว ข้อมูลจะนำไปตรวจสอบกับ API dopa
ถ้าเป็น citizen/idcard จะตรวจสอบกับ request response field firstName หรือ 
ถ้าเป็น citizen/laser จะตรวจสอบกับ request body field firstname
ไม่แน่ใจว่า Service ไหน ลองดูจาก OIC_IIQE_POSTMAN\Bruno\collections\* ประกอบ
แต่ถ้าเป็นกรณีที่ผู้สมัครมีชื่อกลาง (middle name) จะไม่สามรถสมัครสมาชิดได้เอง เพราะว่า
1. หน้าจอยังไม่รองรับชื่อกลางจากการ input ของผู้สมัคร
2. การตรวจสอบกับ api มี property เป็น "ชื่อจริง + ชื่อกลาง" (ชื่อจริง เว้น space 1 ครั้ง ชื่อกลาง) ภาษาไทย

Problems
1. ต้องการให้เพิ่มช่องรองรับชื่อกลาง lblMiddleNameTH, lblMiddleNameEN
2. เอาข้อมูลชื่อกลาง ไปตรวจสอบตามเงื่อนไข (ตรวจสอบแค่จาก TH)
3. lblMiddleNameTH เอาไว้สำหรับตรวจสอบ DOPA อย่างเดียว ไม่ต้องเพิ่ม field หรือ Column เพื่อเก็บข้อมูล
4. การทดสอบ กับ service จะต้องตรวจสอบจาก server UAT/PROD เท่านั้น ใช้ localhost ไม่ได้ อาจจะลำบากหน่อย แต่ผมสามารถ build deploy และแจ้งผลได้
```

```
Context:
Brain (Doc) - D:\Works\OICIIQE\Brain_IIQE
File appsettings production - D:\Works\OICIIQE\Brain_IIQE\appsettings\PROD-2026

Observation:
ระบบ IIQE ตอนที่ Deploy ให้ user ใช้งานจริง แบ่งโครงสร้างเป็น 3 ส่วน
3 Web server (IIS) คือ
- OICIIQE_ONLINE : onlineweb ให้ผู้สมัครใช้งาน
- OICIIQE_INTERNAL : internal  ให้เจ้าหน้าที่ คปภ. ใช้งาน
- APIIIQE : web api เป็ย web api ของระบบ (ส่วนนี้ไม่แน่ใจว่า ให้เฉพาะ online web หรือไม่)

Problems
1. ต้องการให้ตรวจสอบไฟล์ appsettings ทั้ง 3 ส่วน ตาม "File appsettings production"
2. refactor / rearange ให้ดีขึ้น เพื่อง่ายต่อการ maintain
3. แต่ว่าทั้ง 3 ไฟล์นั้น มีการใช้งานจริงอยู่แล้ว และใช้งานได้ ควรจัดอย่างรอบคอบ
```

```
Context:
หน้า /RegisterGeneral/RegisterGeneral เลือกจังหวัดแล้ว อำเภอ ไม่ขึ้น ที่ branch dev
แต่ถ้าสลับกลับไปที่ dev-report ทำงานได้ปกติ

Problems
1. ตรวจสอบ code 2 branch นี้ว่ามีส่วนไหนทำให้กระทบไหม
2. แนวทางการแก้ไข
```

```
Context:
File bruno - D:\Works\OICIIQE\Brain_IIQE\OIC_IIQE_POSTMAN\Bruno\collections\*

Observation:
จากไฟล์ RegisterGeneral.cshtml เป็นหน้าสมัครสมาชิก IIQE
มี field รับข้อมูล lblFirstNameTH, lblLastNameTH เมื่อผู้ใช้กรอกข้อมูลแล้ว ข้อมูลจะนำไปตรวจสอบกับ API dopa
ถ้าเป็น citizen/idcard จะตรวจสอบกับ request response field firstName หรือ 
ถ้าเป็น citizen/laser จะตรวจสอบกับ request body field firstname
ไม่แน่ใจว่า Service ไหน ลองดูจาก OIC_IIQE_POSTMAN\Bruno\collections\* ประกอบ
แต่ถ้าเป็นกรณีที่ผู้สมัครมีชื่อกลาง (middle name) จะไม่สามารถสมัครสมาชิกได้เอง เพราะว่า
1. หน้าจอยังไม่รองรับชื่อกลางจากการ input ของผู้สมัคร
2. การตรวจสอบกับ api มี property เป็น "ชื่อจริง + ชื่อกลาง" (ชื่อจริง เว้น space 1 ครั้ง ชื่อกลาง) ภาษาไทย ลองดูจาก response ของ bruno ที่ save มา ประกอบอีกที 

Problems
1. ต้องการให้เพิ่มช่องรองรับชื่อกลาง lblMiddleNameTH, lblMiddleNameEN
2. เอาข้อมูลชื่อกลาง ไปตรวจสอบตามเงื่อนไข (ตรวจสอบแค่จาก TH)
3. lblMiddleNameTH เอาไว้สำหรับตรวจสอบ DOPA อย่างเดียว ไม่ต้องเพิ่ม field หรือ Column เพื่อเก็บข้อมูล
4. การทดสอบ กับ service จะต้องตรวจสอบจาก server UAT/PROD เท่านั้น ใช้ localhost ไม่ได้ อาจจะลำบากหน่อย แต่ผมสามารถ build deploy และแจ้งผลได้
5. จาก code ปัจจุบัน ต้องปรับอะไรเพิ่มบ้าง สามารถทดสอบได้เลยไหม

เราจะ implement ไปในแนวทางของ v3 เพราะ v2 จะยกเลิการใช้งาน
```

6. appsettings.json จะต้องเพิ่มอะไร

```
Problems:
หน้า /GroupExamRequest/Detail/3028/Edit/3021/2 กดปุ่มส่งคำขอ แล้วไม่ทำงาน ไม่เรียก SaveExamRequest หรือเปล่า ไม่แน่ใจ แต่ถ้า rollback code ไป จะใช้งานได้

Observation:
ตรวจสอบจากการ Merged branch ว่าทำให้ error ไหม และแนวทางการแก้ไข อันนี้เป็น urgent bug ให้ตรวจสอบอย่างละเอียด
```

```
ผมคิดว่า commit 4f599ec365889186bef1e2bcfd78c6e187d396e7 ทำให้ตอนนี้เราเจอปัญหาอื่น ๆ ตามมา เพราะว่า เรา  upgrade .net 6 เป็น .net 8 และ nuget ต่าง ๆ
คิดว่าควรแก้ปัญหาอย่างไรดี
วิธีที่ 1 เอา Commit นี้ออก
วิธีที่ 2 เข้าไปแก้ไขที่ไฟล์ csproj แก้ไข downgrade TargetFramework จาก 8.0 เป็น 6.0 แต่ไม่ต้อง rollback nuget version
```