## OIC IIQE REGISTRATION EXAMINATION SPEC

Context:
- D:\Works\OICIIQE\*
- D:\Works\OICIIQE\OIC_IIQE
- D:\Works\OICIIQE\IIQE_API
- D:\Works\OICIIQE\Brain_IIQE\registration_examination_spec\screenshort\*

ConnectionString:
```json
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"
```

Observation:
ระบบรับสมัครสอบเพื่อขอรับใบอนุญาตเป็นนายหน้าประกันภัย
การสอบรอบปกติบุคคลธรรมดา จะเป็นการเปิดรอบสอบโดยเจ้าหน้าที่ คปภ. และ สมัครสอบโดยประชาชน จะมี Flow ดังนี้

เจ้าหน้าที่ คปภ. เปิดรอบสอบ (สร้างรอบสอบ)
1. เจ้าหน้าที่ คปภ. เข้าสู่ระบบเจ้าหน้าที่ (/Login/LoginStaff)

2. ไปที่หน้า รอบสอบปกติสำหรับสมาชิกบุคคลธรรมดา (/PersonalExamRound/PersonalExamRound) และกดปุ่ม "+ เพิ่ม" ระบบจะ redirect ไปยังหน้า (/PersonalExamRound/PersonalExamRound_New)
<img src="screenshort\1.png" width="75%">
<img src="screenshort\2.png" width="75%">

3. กรอกข้อมูล ดังนี้
- ชื่อศูนย์สอบ (สามารถระบุได้มากกว่า 1 ศูนย์สอบ)
Note: ศูนย์สอบ คือ Table MT_T_EXAM_CENTER
- ประเภทใบอนุญาต
ระบุตามที่ศูนย์สอบที่เลือกที่สามารถเปิดสอบตามประเภทใบอนุญาตนั้น ๆ ได้ เช่น นายหน้าประกันชีวิต, นายหน้าประกันวินาศภัย, อื่น ๆ แต่เลือกได้ 1 ประเภทใบอนุญาต 
Note: ประเภทใบอนุญาต คือ Table MT_T_LICENSE_TYPE
 - นายหน้าประกันชีวิต คือ 
 - นายหน้าประกันวินาศภัย คือ 
- สถานที่สอบ กดปุ่ม "ค้นหาและเลือกสถานที่สอบ"
<img src="screenshort\2.png" width="75%">
จะแสดง Modal "ค้นหาและเลือกสถานที่สอบ"
<img src="screenshort\3.png" width="75%">
<img src="screenshort\4.png" width="75%">
กดบันทึก เพื่อยีนยันสถานที่สอบ
<img src="screenshort\5.png" width="75%">
- ระบุวันที่สอบ และเวลาสอบ
<img src="screenshort\6.png" width="75%">
- กดยืนยันเพื่อบันทึกข้อมูล
<img src="screenshort\7.png" width="75%">

ประชาชน (บุคคลธรรมดา) สมัครสอบ

Problems:
จาก Observation


ประเภทการขึ้นทะเบียน [OICIIQE.MT_T_REGISTRATION_TYPE]
นายหน้าประกันวินาศภัย การประกันต่อ	Life Reinsurance Broker
นายหน้าประกันวินาศภัย การประกันต่อ	Non Life Reinsurance Broker



Task:
1. วางแผนการ Implement ระบบ เพื่อให้รองรับการเปิดรอบสอบ (ใช้ Plan mode)
2. ตรวจสอบผลกระทบ
3. หากมี Script Database ที่จะต้องไป Execute ให้เตรียมให้ด้วย
