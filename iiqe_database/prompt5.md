## Prompt 

## ReportExaminationMember
### Prompt 5.1

```
Context:
Material problem report - D:\Works\OICIIQE\Brain_IIQE\problem_report\ExaminationMember_1

Observation:
หน้า /ReportExaminationMember/Search ระบุเงื่อนไขการค้นหา
หน่วยงานจัดสอบ เป็นหน่วยงานภายนอก, มกราคม 2568 ถึง ธันวาคม 2568, ศูนย์สอบ สมาคมประกันชีวิตไทย, ประเภทใบอนุญาต ตัวแทนประกันชีวิต
ข้อมูลเดือน สิงหาคม ในระบบได้ 10,998 เทียบกับใบส่งมอบงานของสมาคมฯ ได้ 11,004 Diff ที่ 6
ข้อมูลเดือน กันยายน ในระบบได้ 11,725 เทียบกับใบส่งมอบงานของสมาคมฯ ได้ 11,745 Diff ที่ 20
ซึ่งข้อมูลตามใบส่งงานถูกต้อง

Problems
1. ต้องการให้ตรวจสอบไฟล์ appsettings ทั้ง 3 ส่วน ตาม "File appsettings production"
2. refactor / rearange ให้ดีขึ้น เพื่อง่ายต่อการ maintain
3. แต่ว่าทั้ง 3 ไฟล์นั้น มีการใช้งานจริงอยู่แล้ว และใช้งานได้ ควรจัดอย่างรอบคอบ

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"

Task:
1. วิเคราะห์ว่าปัญหานี้เกิดจากอะไร
2. แนวทางการแก้ไขปัญหา
3. แนะนำจุดที่ควรตรวจสอบ เช่น
   - duplicate join
   - cancelled round
   - transfer round
   - absent vs attend
```

```
Context:
- D:\Works\OICIIQE\*
- D:\Works\OICIIQE\OIC_IIQE
- D:\Works\OICIIQE\IIQE_API
- D:\Works\OICIIQE\Brain_IIQE\aaaaaaaaa

Observation:
ระบบรับสมัครสอบเพื่อขอรับใบอนุญาตเป็นนายหน้าประกันภัย
การสอบรอบปกติบุคคลธรรมดา
จะเป็นการเปิดรอบสอบโดยเจ้าหน้าที่ คปภ. และ สมัครสอบโดยประชาชน จะมี Flow ดังนี้
เจ้าหน้าที่ คปภ. เปิดรอบสอบ (สร้างรอบสอบ)
1. เจ้าหน้าที่ คปภ. เข้าสู่ระบบเจ้าหน้าที่ (/Login/LoginStaff)

D:\Works\OICIIQE\Brain_IIQE\registration_examination_spec\1.png
2. ไปที่หน้า รอบสอบปกติสำหรับสมาชิกบุคคลธรรมดา (/PersonalExamRound/PersonalExamRound) และกดปุ่ม "+ เพิ่ม" ระบบจะ redirect ไปยังหน้า (/PersonalExamRound/PersonalExamRound_New)
3. กรอกข้อมูล ดังนี้
- ชื่อศูนย์สอบ (สามารถระบุได้มากกว่า 1 ศูนย์สอบ)
Note: ศูนย์สอบ คือ Table MT_T_EXAM_CENTER
- ประเภทใบอนุญาต ระบุตามที่ศูนย์สอบที่เลือกที่สามารถเปิดสอบตามประเภทใบอนุญาตนั้น ๆ ได้ เช่น นายหน้าประกันชีวิต, นายหน้าประกันวินาศภัย, อื่น ๆ แต่เลือกได้ 1 ประเภทใบอนุญาต 
Note: ประเภทใบอนุญาต คือ Table MT_T_LICENSE_TYPE
นายหน้าประกันชีวิต
นายหน้าประกันวินาศภัย
D:\Works\OICIIQE\Brain_IIQE\registration_examination_spec\2.png
- สถานที่สอบ กดปุ่ม "ค้นหาและเลือกสถานที่สอบ"
จะแสดง Modal 

ประชาชน (บุคคลธรรมดา) สมัครสอบ

Problems:
จาก Observation


ประเภทการขึ้นทะเบียน [OICIIQE.MT_T_REGISTRATION_TYPE]
นายหน้าประกันวินาศภัย การประกันต่อ	Life Reinsurance Broker
นายหน้าประกันวินาศภัย การประกันต่อ	Non Life Reinsurance Broker

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"

Task:
1. วางแผนการ Implement ระบบ เพื่อให้รองรับการเปิดรอบสอบ (ใช้ Plan mode)
2. ตรวจสอบผลกระทบ
3. หากมี Script Database ที่จะต้องไป Execute ให้เตรียมให้ด้วย
```
