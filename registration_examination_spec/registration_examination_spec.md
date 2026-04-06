# OIC IIQE Registration Examination Spec

> การสอบการขึ้นทะเบียน นายหน้าประกันชีวิต/นายหน้าประกันวินาศภัย การประกันต่อ Life/Non Life Reinsurance Broker

---

## Context

- `D:\Works\OICIIQE\*`
- `D:\Works\OICIIQE\OIC_IIQE`
- `D:\Works\OICIIQE\IIQE_API`
- `D:\Works\OICIIQE\Brain_IIQE\registration_examination_spec\screenshort\*`

---

## Links

- [[IIQE 2025] ประกันภัยต่อ สรุป](https://docs.google.com/spreadsheets/d/11idsezbjxl3TPpq6DsTspX2MMYa8NEnIhEyejgE5UnU/)
- [[IIQE 2025] ประกันภัยต่อ](https://docs.google.com/spreadsheets/d/1a2PSQeYnBNX1KnhCa-XqZv8l_G1LiwVf5LqQssVFeVE/)

---

## Connection String

```json
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"
```

---

## Observation

ระบบรับสมัครสอบเพื่อขอรับใบอนุญาตเป็นนายหน้าประกันภัย  
การสอบรอบปกติบุคคลธรรมดา จะเป็นการเปิดรอบสอบโดยเจ้าหน้าที่ คปภ. และ สมัครสอบโดยประชาชน (บุคคลธรรมดา) จะมี Flow ดังนี้

> ตัวอย่างนี้จะเป็นใบอนุญาต **นายหน้าประกันชีวิต**

### 1. เจ้าหน้าที่ คปภ. เปิดรอบสอบ (สร้างรอบสอบ)

1. เจ้าหน้าที่ คปภ. เข้าสู่ระบบเจ้าหน้าที่ (`/Login/LoginStaff`)

2. ไปที่หน้า **รอบสอบปกติสำหรับสมาชิกบุคคลธรรมดา** (`/PersonalExamRound/PersonalExamRound`) และกดปุ่ม **"+ เพิ่ม"** ระบบจะ redirect ไปยังหน้า (`/PersonalExamRound/PersonalExamRound_New`)

   <img src="screenshort/1.png" alt="หน้ารายการรอบสอบ" style="border: 1px solid black; width: 100%;">

   > **[รูป 1]** หน้ารายการรอบสอบปกติสำหรับสมาชิกบุคคลธรรมดา (`/PersonalExamRound/PersonalExamRound`)
   > แสดง filter ค้นหา (ชื่อศูนย์สอบ, ชื่อสถานที่สอบ, ประเภทใบอนุญาต, ช่วงวันที่, เวลาสอบ, รหัสรอบสอบ, สถานะ)
   > และตารางรายการรอบสอบที่มีอยู่ โดยมีปุ่ม **"+ เพิ่ม"** สำหรับเพิ่มรอบสอบใหม่
   > คอลัมน์ในตาราง: ลำดับ, ชื่อสถานที่สอบ, วันที่สอบ, เวลาสอบ, ประเภทใบอนุญาต, รหัสรอบสอบ, ที่นั่งสอบทั้งหมด, ที่นั่งคงเหลือ, สถานะ, แก้ไขล่าสุดโดย, วันที่/เวลาแก้ไขล่าสุด

   <img src="screenshort/2_1.png" alt="หน้าเพิ่มรอบสอบใหม่" style="border: 1px solid black; width: 100%;">

   > **[รูป 2]** หน้าฟอร์มสร้างรอบสอบใหม่ (`/PersonalExamRound/PersonalExamRound_New`)
   > แสดง field กรอกข้อมูล ได้แก่:
   > - **ชื่อศูนย์สอบ** (แบบ multi-select tag): ตัวอย่าง "สำนักงาน คปภ. ภาค 7 (นครปฐม)"
   > - **ประเภทใบอนุญาต** (dropdown): ตัวอย่าง "นายหน้าประกันชีวิต"
   > - **สถานที่สอบ**: ปุ่ม "ค้นหาและเลือกสถานที่สอบ"
   > - **วันที่สอบ**: ปฏิทิน แสดงเดือน/ปี (เม.ย. 2569) พร้อม checkbox เลือกวันที่ต้องการ (เสาร์-อาทิตย์ highlight สีชมพู)

3. กรอกข้อมูล ดังนี้

   - **ชื่อศูนย์สอบ** (สามารถระบุได้มากกว่า 1 ศูนย์สอบ)  
     > **Note:** ศูนย์สอบ คือ Table `MT_T_EXAM_CENTER`

   - **ประเภทใบอนุญาต**  
     ระบุตามที่ศูนย์สอบที่เลือกที่สามารถเปิดสอบตามประเภทใบอนุญาตนั้น ๆ ได้ เช่น นายหน้าประกันชีวิต, นายหน้าประกันวินาศภัย, อื่น ๆ แต่เลือกได้ 1 ประเภทใบอนุญาต  
     > **Note:** ประเภทใบอนุญาต คือ Table `MT_T_LICENSE_TYPE`
     - นายหน้าประกันชีวิต
     - นายหน้าประกันวินาศภัย

   - **สถานที่สอบ** กดปุ่ม **"ค้นหาและเลือกสถานที่สอบ"**

     <img src="screenshort/2_1.png" alt="ปุ่มค้นหาสถานที่สอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 2 - ซ้ำ]** แสดงปุ่ม **"ค้นหาและเลือกสถานที่สอบ"** อยู่ในแถว field สถานที่สอบ บนหน้าฟอร์มสร้างรอบสอบ

     ระบบจะแสดง Modal **"ค้นหาและเลือกสถานที่สอบ"**

     <img src="screenshort/3_1.png" alt="Modal ค้นหาสถานที่สอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 3]** Modal **"ค้นหาและเลือกสถานที่สอบ"** (ยังไม่ได้ค้นหา)
     > มี filter: **กลุ่มสถานที่สอบ** (dropdown: "-- กรุณาเลือก --") และ **จังหวัด** (dropdown: "นครปฐม")
     > ปุ่ม **"เลือก"** สำหรับค้นหา, ตารางแสดงผลมีคอลัมน์: เลือก, ลำดับ, ชื่อสถานที่สอบ, จังหวัด, กลุ่มสถานที่สอบ, ประเภท
     > สถานะ: "ไม่พบข้อมูล" (เพราะยังไม่ได้กดค้นหา)

     <img src="screenshort/4_1.png" alt="Modal ผลการค้นหาสถานที่สอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 4]** Modal **"ค้นหาและเลือกสถานที่สอบ"** (หลังค้นหาแล้ว)
     > filter: กลุ่มสถานที่สอบ = "สำนักงาน คปภ.", จังหวัด = "นครปฐม"
     > ผลการค้นหา: พบ 1 รายการ — ลำดับ 1: สถานที่สอบ "73777 สำนักงาน คปภ. ภาค 7 (นครปฐม)", จังหวัด: นครปฐม, กลุ่ม: สำนักงาน คปภ., ประเภท: ในความรับผิดชอบของสำนักงาน คปภ.
     > มี checkbox เลือก (ถูก tick แล้ว)

     > **Note:** สถานที่สอบ คือ Table `MT_T_EXAM_LOCATION`

     กดบันทึก เพื่อยืนยันสถานที่สอบ

     <img src="screenshort/5_1.png" alt="ยืนยันสถานที่สอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 5]** หน้าฟอร์มหลัง บันทึกสถานที่สอบสำเร็จ
     > field **สถานที่สอบ** แสดง card: checkbox tick + รหัส "73777" + ชื่อ "สำนักงาน คปภ. ภาค 7 (นครปฐม)" พร้อมปุ่ม ✕ สำหรับลบ
     > ด้านล่างยังแสดง section เลือกวันที่สอบ (ปฏิทิน เม.ย. 2569)

   - **ระบุวันที่สอบ และเวลาสอบ**

     <img src="screenshort/6_1.png" alt="กรอกวันที่และเวลาสอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 6]** Section กรอกวันที่และเวลาสอบ
     > - **วันที่สอบ**: ปฏิทิน เดือน เม.ย. ปี 2569 — เลือกวันที่ 17 (ศุกร์) ไว้แล้ว (checkbox สีน้ำเงิน)
     > - **เวลาสอบ**: multi-tag เลือกได้หลายช่วง ตัวอย่าง "09:00-11:00" และ "12:00-14:30"
     > - **ค่า Default วันเปิด/ปิดรับสมัครสอบ**: รูปแบบที่ 1 — เปิดรับสมัครก่อนวันสอบ 20 วัน, ปิดรับสมัครก่อนวันสอบ 7 วัน (พร้อมปุ่ม reset)
     > - **สถานะ**: radio button (เปิดการใช้งาน / ปิดการใช้งาน)
     > - ปุ่ม **"บันทึก"** และ **"ปิด"**

   - กดยืนยันเพื่อบันทึกข้อมูลรอบสอบ

     <img src="screenshort/7.png" alt="ยืนยันบันทึกรอบสอบ" style="border: 1px solid black; width: 100%;">

     > **[รูป 7]** Dialog ยืนยันการบันทึกสำเร็จ
     > แสดง icon วงกลมสีเขียว + เครื่องหมายถูก พร้อมข้อความ **"บันทึกข้อมูลเรียบร้อยแล้ว"** และปุ่ม **"ปิด"**
     > แสดงทับบน background ของหน้าฟอร์ม (เบลอ)

     > **Note:** รอบสอบ คือ Table `MT_T_P_EXAM_ROUND` และ Table `MT_T_P_EXAM_ROUND_DT`

4. ระบบแสดงรอบสอบที่สร้าง

   <img src="screenshort/8_2.png" alt="หน้ารายการรอบสอบ แสดงรูปรอบสอบที่สร้าง" style="border: 1px solid black; width: 100%;">

   > **[รูป 8_2]** หน้ารายการรอบสอบปกติ (`/PersonalExamRound/PersonalExamRound`) หลังสร้างรอบสอบสำเร็จ
   > แสดงรายการรอบสอบที่มีอยู่ในระบบ โดยรอบสอบคือที่อยู่ลำดับ 3 และ 4
   > ข้อมูลรอบสอบที่สร้างใหม่: สถานที่ "สำนักงาน คปภ. ภาค 7 (นครปฐม)", วันที่ 17/04/2569, ประเภท "นายหน้าประกันชีวิต"
   > รหัสรอบสอบ: 6910300263 (13:00-15:30) และ 6910300262 (09:00-11:30), ที่นั่ง 10 ที่, สถานะ "เปิดใช้งาน" (toggle สีเขียว)


### 2. ประชาชน (บุคคลธรรมดา) สมัครสอบ

1. ประชาชน (บุคคลธรรมดา) เข้าสู่ระบบสำหรับประชาชน (`/Login/Login`) ระบบจะ Redirect ไปหน้า Dashboard-หน้าหลัก (`/TaskBoards/Task`)

   <img src="screenshort/9.png" alt="หน้า Dashboard-หน้าหลัก" style="border: 1px solid black; width: 100%;">

   > **[รูป 9]** หน้า **Dashboard-หน้าหลัก** (`/TaskBoards/Task`) สำหรับผู้ใช้งานประชาชน (บุคคลธรรมดา)
   > แสดง widget สรุปสถานะคำขอ แบ่งเป็น 4 กลุ่ม:
   > - **คำขอสมัครสอบรอบปกติ**: รอดำเนินการ (ร่าง, ส่งคำขอ-รอจัดทำใบแจ้งชำระเงิน, รอชำระเงิน, ชำระเงินแล้ว)
   > - **คำขอสมัครสอบรอบพิเศษ**: รอดำเนินการ (รอเข้าสอบ)
   > - **ประวัติคำขอสมัครสอบรอบปกติ**: ผลการสอบ (รออนุมัติ, อนุมัติแล้ว), ประวัติคำขอ (คำขอที่จบกระบวนการแล้ว)
   > - **ประวัติคำขอสมัครสอบรอบพิเศษ**: ผลการสอบ, ประวัติคำขอ
   > เมนูบาร์: ตารางสอบ, คำขอสมัครสอบรอบปกติ, ประวัติและผลสอบรอบปกติ, คำขอสมัครสอบรอบพิเศษ, ประวัติและผลสอบรอบพิเศษ, แจ้งปัญหาการเข้าใช้งาน, FAQ, ข้อมูลสมาชิก

   > **Note:** ข้อมูลผู้ใช้งาน/สมาชิก
   > - `MT_T_USER` — ข้อมูล Login (USERNAME, PASSWORD, ROLE_TYPE_ID, PERSONAL_MEMBER_ID)
   > - `MT_T_PERSONAL_MEMBER` — ข้อมูลสมาชิกบุคคลธรรมดา (FIRST_NAME_TH, LAST_NAME_TH, ID_CARD, EMAIL, MOBILE_PHONE, STATUS)
   > - `MT_T_ROLE_TYPE` — ประเภท Role และ DEFAULT_MENU_URL (ROLE_TYPE_ID = 1 = บุคคลธรรมดา)

2. กดปุ่ม **ตารางสอบ** ที่แถบเมนู เพื่อเข้าสู่หน้าตารางสอบ (`/ExamSchedule/ExamSchedule`)

   <img src="screenshort/10.png" alt="หน้าตารางสอบ (ก่อนค้นหา)" style="border: 1px solid black; width: 100%;">

   > **[รูป 10]** หน้า **ตารางสอบ** (`/ExamSchedule/ExamSchedule`) ก่อนค้นหา
   > มี filter การค้นหา (ทุก field บังคับ): **ประเภทใบอนุญาต**, **จังหวัด**, **สถานที่สอบ/สนามสอบ**, **ช่วงวันที่สอบ ตั้งแต่วันที่**, **ถึงวันที่**
   > มีหมายเหตุสีแดง: 1. ระบบเปิดรับสมัครสอบก่อนวันสอบ 20 วัน และปิดรับสมัครก่อนวันสอบ 7 วัน / 2. การค้นหาสถานที่สอบ "มหาวิทยาลัยธรรมศาสตร์ ศูนย์รังสิต" ให้เลือกจังหวัด "ปทุมธานี"
   > ปุ่ม: **ค้นหา**, **เคลียร์**, **ปิด**

   > **Note:** ข้อมูลที่ใช้แสดงหน้าตารางสอบ
   > - `MT_T_P_EXAM_ROUND` — รอบสอบ (EXAM_DATE, EXAM_TIME_ID, EXAM_LOCATION_ID, LICENSE_TYPE_ID, ADMISSION_S_DATE, ADMISSION_E_DATE, C_ROUND_SETTING_MAX_AMOUNT, IS_ACTIVE)
   > - `MT_T_EXAM_TIME` — เวลาสอบ (TEST_TIME เช่น "09:00-11:30")
   > - `MT_T_EXAM_LOCATION` — สถานที่สอบ (NAME_TH, AG_CODE, CODE, PROVINCE_ID)
   > - `MT_T_LICENSE_TYPE` — ประเภทใบอนุญาต (CODE, DISPLAY_NAME_TH)
   > - `MT_T_PROVINCE` — จังหวัด (NAME_TH) ใช้ใน filter ค้นหา
   > - `IIQE_T_P_EXAM_ROUND_LIST` — นับจำนวนที่นั่งที่มีผู้สมัครแล้ว (COUNT โดย EXAM_SEAT_NUMBER IS NOT NULL)
   
3. ค้นหารอบสอบโดยระบุ Parameter

   - ประเภทใบอนุญาต: นายหน้าประกันชีวิต
   - จังหวัด: นครปฐม
   - สถานที่สอบ/สนามสอบ: สำนักงาน คปภ. ภาค 7 (นครปฐม)
   - ช่วงวันที่สอบ ตั้งแต่วันที่: 05/04/2569
   - ถึงวันที่: 30/04/2569

   <img src="screenshort/11_1.png" alt="หน้าตารางสอบหลังค้นหา" style="border: 1px solid black; width: 100%;">

   > **[รูป 11_1]** หน้า **ตารางสอบ** หลังค้นหาด้วย filter ที่ระบุ
   > แสดงผลในรูปแบบ **ปฏิทิน** (calendar view) แบ่งตามคอลัมน์ วันจันทร์-อาทิตย์
   > Legend: สีน้ำเงิน = วันที่สอบที่เปิดรับสมัครอยู่, สีเทา = วันที่สอบที่ปิดรับสมัครแล้ว
   > รอบสอบที่แสดง (ประเภท "นายหน้าประกันชีวิต"):
   > - **10 เม.ย. 2569 (ศุกร์)**: สำนักงาน คปภ. ภาค 7 (นครปฐม) — 09:00-11:30 น., 12:00-14:30 น.
   > - **17 เม.ย. 2569 (ศุกร์)**: สำนักงาน คปภ. ภาค 7 (นครปฐม) — 09:00-11:30 น. (สีน้ำเงิน/เปิดรับสมัคร), 13:00-15:30 น.

4. กดเลือกเวลาสอบจากหน้าตารางสอบ เช่น เลือกเวลา **"09:00-11:30 น. ของวันที่ 17 เม.ย. 2569"** ระบบจะ redirect ไปยังหน้า **คำขอสมัครสอบรอบปกติสำหรับสมาชิกบุคคลธรรมดา** (`/ExamRequest/Detail/New/0/31797`) พร้อมระบุข้อมูลรอบสอบให้อัตโนมัติตามที่ผู้สมัครเลือก หากข้อมูลถูกต้อง ให้กดปุ่ม **"สร้างคำขอสมัครสอบ"**

   <img src="screenshort/12.png" alt="หน้าคำขอสมัครสอบรอบปกติ" style="border: 1px solid black; width: 100%;">

   > **[รูป 12]** หน้า **คำขอสมัครสอบรอบปกติสำหรับสมาชิกบุคคลธรรมดา** (`/ExamRequest/Detail/New/0/31797`)
   > ส่วนบน: ข้อมูลที่อยู่ของผู้สมัคร (ที่อยู่, ตำบล/แขวง, อำเภอ/เขต, จังหวัด, รหัสไปรษณีย์)
   > ส่วน **"รอบสอบที่สมัครสอบ"**: มีฟังก์ชันค้นหารอบสอบ (ประเภทใบอนุญาต, จังหวัด, สถานที่สอบ) และแสดงข้อมูลรอบสอบที่เลือก:
   > - ประเภทใบอนุญาต: นายหน้าประกันชีวิต
   > - รหัสรอบสอบ: 6910300262, วันที่สอบ: 17/04/2569, เวลาสอบ: 09:00-11:30
   > - จังหวัด: นครปฐม, สถานที่สอบ: สำนักงาน คปภ. ภาค 7 (นครปฐม)
   > - ชื่อศูนย์สอบที่รับผิดชอบ: สำนักงาน คปภ. ภาค 7 (นครปฐม)
   > ปุ่ม: **"บันทึกร่าง"**, **"สร้างคำขอสมัครสอบ"** (สีเขียว), **"ปิด"**

   - ระบบแสดง Popup ยืนยัน ถ้ากด **"ยืนยัน"** จะสร้างคำขอสมัครสอบ

   <img src="screenshort/13.png" alt="Popup ยืนยันสร้างคำขอสมัครสอบ" style="border: 1px solid black; width: 100%;">

   > **[รูป 13]** Dialog ยืนยันการสร้างคำขอสมัครสอบ
   > แสดง icon วงกลมสีเขียว + เครื่องหมาย **?** พร้อมข้อความ **"ยืนยันการสร้างคำขอสมัครสอบหรือไม่?"**
   > ปุ่ม: **"ยืนยัน"** (สีน้ำเงิน) และ **"ยกเลิก"** (สีเทา)
   > แสดงทับบน background ของหน้าคำขอสมัครสอบ (เบลอ) ข้อมูลรอบสอบยังคงมองเห็นอยู่เบื้องหลัง

   > **Note:** การสร้างคำขอสมัครสอบจะ Insert/Update ลง Tables ดังนี้
   > - `IIQE_T_P_EXAM_REQ` — คำขอสมัครสอบหลัก (PERSONAL_MEMBER_ID, P_EXAM_ROUND_ID, EXAM_FEE, REQ_DATE, FLOW_STATUS_ID, RUNNING_NUMBER, RUNNING_YEAR, RUNNING_MONTH)
   > - `IIQE_T_P_EXAM_ROUND_LIST` — จองที่นั่งสอบ (P_EXAM_REQ_ID, P_EXAM_ROUND_ID, EXAM_SEAT_NUMBER, EXAM_LOCATION_ID)
   > - `IIQE_T_P_EXAM_REQ_FLOW` — ประวัติการเปลี่ยน Status คำขอ (P_EXAM_REQ_ID, FLOW_STATUS_ID)
   > - `MT_T_FLOWSTATUS` — ชื่อสถานะ (ID: 6 = ร่าง, 46 = ส่งคำขอ/รอจัดทำใบแจ้งชำระเงิน, 47 = รอชำระเงิน, 49 = ชำระเงินแล้ว)

5. ระบบ redirect ไปยังหน้า **ชำระเงินค่าสมัครสอบ**

   <img src="screenshort/14.png" alt="หน้าชำระเงินค่าสมัครสอบ" style="border: 1px solid black; width: 100%;">

   > **[รูป 14]** หน้า **คำขอสมัครสอบรอบปกติสำหรับสมาชิกบุคคลธรรมดา** (`/ExamRequest/Detail/Edit/12`) หลังสร้างคำขอสำเร็จ
   > แสดง Stepper workflow 3 ขั้นตอน: **1. สร้างคำขอสมัครสอบรอบปกติ** (เขียว/เสร็จแล้ว) → **2. ชำระเงินค่าสมัครสอบ** (น้ำเงิน/ขั้นตอนปัจจุบัน) → ผลการสอบ
   > ข้อมูลที่แสดง:
   > - เลขที่คำขอ: 69041000002, วันที่ยื่นคำขอ: 05/เม.ย./2569
   > - ประเภทใบอนุญาต: นายหน้าประกันชีวิต
   > - รหัสรอบสอบ: 6910300263, วันที่สอบ: 17/เม.ย./2569
   > - สถานที่สอบ: สำนักงาน คปภ. ภาค 7 (นครปฐม)
   > ปุ่ม: **"ส่งคำขอสมัครสอบและออกใบแจ้งการชำระเงิน"** (สีน้ำเงิน), **"ยกเลิกคำขอสมัครสอบ"** (สีแดง), **"ปิด"**

   > **Note:** ข้อมูลที่แสดงในหน้านี้มาจาก `IIQE_T_P_EXAM_REQ` (JOIN กับ `MT_T_P_EXAM_ROUND`, `MT_T_EXAM_LOCATION`, `MT_T_LICENSE_TYPE`)
   > การตั้งค่าวันครบกำหนดชำระเงินมาจาก `MT_T_PAYMENT_DUEDATE` (NUMBER_OF_DAYS, TYPE, EXAM_TYPE)
   > ระบบตรวจสอบ Blocklist จาก `MT_T_EXAM_FRAUD_LIST` และ `LOG_T_BLOCKLIST_LOG` ก่อนอนุญาตให้ดำเนินการต่อ

6. ผู้สมัครกดปุ่ม ส่งคำขอสมัครสอบและออกใบแจ้งการชำระเงิน ระบบจะทำการส่งข้อมูลการสมัครสอบของผู้สมัคร ไปยังระบบออกใบแจ้งการชำระเงิน (ฺBillpayment) (ระบบ ERP เป็น Microsoft Dynamic 365) ผ่าน Web API เมื่อระบบ ERP สร้าง Bill Payment เรียบร้อยแล้ว จะส่งข้อมูลกลับมายังระบบ OICIIQE
ข้อมูลส่งไปสร้าง Billpayment จะมีดังนี้

   **ข้อมูลผู้สมัคร (Customer) — ส่งไปยัง `InImpCustomer`:**
   - รหัสสมาชิก (`CUST_ID`): `MEMBER_CODE`
   - ชื่อ-นามสกุล (`CUST_NAME`): ชื่อเต็มภาษาไทยของผู้สมัคร
   - เบอร์โทรศัพท์ (`TEL_ID`): `Mobile_Phone`
   - อีเมล (`EMAIL_ID`): อีเมลของสมาชิก
   - ที่อยู่ (`CUST_ADDR1`): ADDRESS + ตำบล/แขวง + อำเภอ/เขต + จังหวัด
   - รหัสไปรษณีย์ (`POST_ID`): POST_CODE (padding 5 หลัก)
   - ประเภทลูกค้า (`CUST_TYPE`): `"1"` (บุคคลธรรมดา)
   - กลุ่มลูกค้า (`GROUP_ID`): `"AR03"`

   **ข้อมูลใบแจ้งชำระเงิน (Bill Payment) — ส่งไปยัง `InImpMainRevenue`:**
   - เลขที่คำขอสมัครสอบ (`REFERENCE_NO`): `P_EXAM_REQ_CODE`
   - เลข Reference 1 (`REFERENCE_1`): คำนวณจาก Prefix + วันที่สอบ (ddMM) + ปี พ.ศ. (ตัวอย่าง: `0002512567`)
   - เลข Reference 2 (`REFERENCE_2`): คำนวณจาก Checksum ของ Ref1 + ยอดชำระ
   - ยอดค่าธรรมเนียมสอบ (`NET_AMT` / `TOT_AMT`): 200 บาท
   - วันครบกำหนดชำระ (`DUE_DT`): คำนวณจาก Bill Payment Setting
   - Biller ID (`BILLER_ID`): จาก Bill Payment Setting
   - รหัสบริษัท (`COMP_CODE`): `"OIC"`
   - รหัสสินค้า (`ITEMID`): จากฐานข้อมูล
   - ชื่อค่าธรรมเนียม (`DESCP_1`): `FEE_NAME`
   - รายละเอียดรอบสอบ (`DESCP_2`): `"วันที่สอบ dd/MMM/yyyy เวลาสอบ HH:mm สถานที่สอบ ..."`
   - วันที่ต้องชำระ (`DESCP_3`): `"* ผู้สมัครสอบต้องชำระเงินภายใน 20.00 น. ของวันที่ dd/MMM/yyyy"`
   - Cost Center (`FD02_CostCenter`): `"231000"`
   - Posting Profile: `"EXAM"`
   - Bill Flag (`BillFlag`): `"Y"`

   หลังจากนั้นระบบ IIQE จะรับ Response จาก ERP และบันทึกลง Table `IIQE_T_BILL_PAYMENT` โดยแสดงข้อมูลดังนี้

   | ข้อมูลที่แสดง | Field ใน `IIQE_T_BILL_PAYMENT` | Field จาก ERP Response |
   |---|---|---|
   | เลข Ref.1 | `REF_NO_1` | `Description[0].REFERENCE1` |
   | เลข Ref.2 | `REF_NO_2` | `Description[0].REFERENCE2` |
   | เลขที่ใบแจ้งชำระ | `REF_NO` | `Description[0].REFERENCENO` |
   | ยอดที่ต้องชำระ (200 บาท) | `AMOUNT` | ค่าที่ส่งไป (`NET_AMT`) ไม่ได้รับกลับจาก ERP |
   | วันที่ครบกำหนดชำระ | `DUE_DATE` | `Description[0].DUE_DT` |
   | QR Code สำหรับ Scan เพื่อชำระเงิน | `QR_CODE` | สร้างจาก `\|BILLER_ID + REF_NO_1 + REF_NO_2 + AMOUNT` |
   | PDF ใบแจ้งชำระเงิน | `DOWNLOAD_ATTACHMENT_ID` → `MT_T_DOWNLOAD_ATTACHMENT.KEY` | `Description[0].FILE_DB64` (base64 → upload → เก็บ key) |
   | Biller ID | `BILLER_ID` | จาก Bill Payment Setting |
   | Company Code | `COMPANY_CODE` | `"OIC"` |
   | สถานะ | `STATUS` | `1` = รอชำระเงิน |
   | ประเภทสมาชิก | `MEMBER_TYPE` | `1` = บุคคลธรรมดา |

   > **Note:** Log การติดต่อ ERP บันทึกไว้ที่ Table `IIQE_LOG_BILLPAYMENT` (field: `REF_1`, `REF_2`, `DATA_JSON_REQ`, `DATA_JSON_RES`, `STATUS`, `RESPONSE_MESSAGE`)
  
7. ผู้สมัครดำเนินการชำระเงินผ่านชื่อทางต่าง ๆ ที่กำหนด เมื่อธนาคารได้รับข้อมูลการชำระเงิน จะส่งข้อมูลการชำระเงิน ไปยังระบบ ERP แล้วระบบ ERP ประมวลผลการชำระเงิน แล้วส่งข้อมูลใบเสร็จ (Receipt) กลับมายังระบบ OICIIQE

   > **Note:** ข้อมูลการชำระเงินและใบเสร็จ
   > - `IIQE_T_PAYMENT_DETAIL` — รายการชำระเงินจากธนาคาร (P_EXAM_REQ_ID, BANK_CODE, PAYMENT_DATE, PAYMENT_TIME, REF_1, REF_2, AMOUNT, RECEIPT_NO)
   > - `IIQE_T_PAYMENT_HEADER` — Header ของ batch file ที่รับจากธนาคาร (FILE_NAME, SERVICE_CODE, EFFECTIVE_DATE, IMPORT_BY)
   > - `IIQE_T_RECEIPT` — ใบเสร็จจาก ERP (REF_NO_1, REF_NO_2, P_EXAM_REQ_ID, CODE, RECEIPT_DATE, TOTAL_AMOUNT, URL_PATH, ERP_RESULT)
   > - `IIQE_T_RECEIPT_QUEUE` — คิวประมวลผลใบเสร็จ (batch processing)
   >
   > **การเชื่อมโยงกับ `IIQE_T_BILL_PAYMENT` และ `MT_T_DOWNLOAD_ATTACHMENT`:**
   >
   > เมื่อ ERP ส่ง Receipt กลับมา ระบบจะดำเนินการ 3 ขั้นตอนตามลำดับ:
   >
   > **ขั้นที่ 1 — Insert ใบเสร็จ** บันทึกลง `IIQE_T_RECEIPT`
   > JOIN กับ `IIQE_T_BILL_PAYMENT` ผ่าน `P_EXAM_REQ_ID` เพื่อดึง `REF_CODE` (เลขที่คำขอ) มาแสดงในรายงาน
   >
   > **ขั้นที่ 2 — Upload PDF ใบเสร็จ** PDF (base64 จาก ERP) → upload ไปยัง File Server
   > บันทึก file key ลง `MT_T_DOWNLOAD_ATTACHMENT` (ID, MENU_ID = 3, KEY = file path)
   > แล้ว Update `IIQE_T_BILL_PAYMENT.DOWNLOAD_ATTACHMENT_ID` ให้ชี้ไปยัง record ใน `MT_T_DOWNLOAD_ATTACHMENT`
   >
   > **ขั้นที่ 3 — Update สถานะ** `IIQE_T_BILL_PAYMENT.STATUS = 3` (ชำระเงินแล้ว/มีใบเสร็จ) match จาก `P_EXAM_REQ_ID`
   >
   > **Relationship:**
   > ```
   > IIQE_T_RECEIPT.P_EXAM_REQ_ID
   >     └──→ IIQE_T_BILL_PAYMENT.P_EXAM_REQ_ID
   >               └──→ IIQE_T_BILL_PAYMENT.DOWNLOAD_ATTACHMENT_ID
   >                         └──→ MT_T_DOWNLOAD_ATTACHMENT.ID
   >                                   └── .KEY = file path ของ PDF ใบแจ้งชำระ/ใบเสร็จ
   > ```

8. กระบวนการสมัครสอบเสร็จสิ้น ผู้สมัครสอบมีสิทธิ์เข้าสอบตามข้อมูลที่สมัครสอบ (รอบสอบ, สถานที่สอบ, วันที่สอบ, เวลาสอบ)

   > **Note:** หลังชำระเงินสำเร็จ ระบบจะ Update `IIQE_T_P_EXAM_REQ.FLOW_STATUS_ID` และ Insert ประวัติการเปลี่ยนสถานะลง `IIQE_T_P_EXAM_REQ_FLOW`
   > ผู้สมัครจะมีข้อมูลที่นั่งสอบใน `IIQE_T_P_EXAM_ROUND_LIST.EXAM_SEAT_NUMBER`

---

## Information
### Tables MT_T_LICENSE_TYPE : ประเภทใบอนุญาต

| ลำดับ | Column Name | Data Type | Allow Nulls | Field Description |
|---|---|---|:---:|---|
| 1 | ID | NUMBER(10,0) | Y | ID |
| 2 | AGENT_TYPE_ID | NUMBER(10,0) | Y | FK รหัสประเภทตัวแทน (ปัจจุบัน NULL ทั้งหมด) |
| 3 | INSURANCE_TYPE_ID | NUMBER(10,0) | Y | FK รหัสประเภทการประกันภัย (ปัจจุบัน NULL ทั้งหมด) |
| 4 | BROKER_TYPE_ID | NUMBER(10,0) | Y | FK รหัสประเภทนายหน้า (ปัจจุบัน NULL ทั้งหมด) |
| 5 | CODE | VARCHAR2(50) | Y | รหัสประเภทใบอนุญาต เช่น `03` = นายหน้าประกันชีวิต, `04` = นายหน้าประกันวินาศภัย |
| 6 | NAME_TH | VARCHAR2(250) | Y | ชื่อประเภทใบอนุญาต (ภาษาไทย) |
| 7 | NAME_EN | VARCHAR2(250) | Y | ชื่อประเภทใบอนุญาต (ภาษาอังกฤษ) |
| 8 | IS_ACTIVE | NUMBER(1,0) | Y | สถานะการใช้งาน: `1` = เปิดใช้งาน, `0` = ปิดใช้งาน |
| 9 | CREATE_BY | VARCHAR2(50) | Y | ผู้สร้างข้อมูล |
| 10 | CREATE_DATE | TIMESTAMP(6) | Y | วันที่/เวลาสร้างข้อมูล |
| 11 | UPDATE_BY | VARCHAR2(50) | Y | ผู้แก้ไขข้อมูลล่าสุด |
| 12 | UPDATE_DATE | TIMESTAMP(6) | Y | วันที่/เวลาแก้ไขข้อมูลล่าสุด |
| 13 | FIN_CODE | VARCHAR2(20) | Y | รหัสประเภทใบอนุญาตสำหรับระบบการเงิน เช่น `203`, `204` |
| 14 | ERP_CODE | VARCHAR2(50) | Y | รหัสสำหรับระบบ ERP (ปัจจุบัน NULL ทั้งหมด) |
| 15 | ERP_NAME | VARCHAR2(250) | Y | ชื่อสำหรับระบบ ERP (ปัจจุบัน NULL ทั้งหมด) |
| 16 | FEE_NAME | VARCHAR2(500) | Y | ชื่อค่าธรรมเนียมสอบที่ใช้แสดงในใบแจ้งชำระเงิน เช่น `ค่าธรรมเนียมสมัครสอบนายหน้าประกันชีวิต ประเภทการประกันชีวิตโดยตรง` |
| 17 | TRANSFER_STATUS | NUMBER(1,0) | Y | สถานะ Transfer ข้อมูลไปยังระบบ AG: `0` = รอ Transfer, `1` = Transfer แล้ว |
| 18 | TRANSFER_DATE | TIMESTAMP(6) | Y | วันที่/เวลาที่ Transfer ข้อมูลไปยังระบบ AG |
| 19 | DISPLAY_NAME_TH | VARCHAR2(250) | Y | ชื่อที่ใช้แสดงบนหน้าจอ (ภาษาไทย) อาจต่างจาก NAME_TH |
| 20 | DISPLAY_NAME_EN | VARCHAR2(250) | Y | ชื่อที่ใช้แสดงบนหน้าจอ (ภาษาอังกฤษ) อาจต่างจาก NAME_EN |
| 21 | OIC_CODE | VARCHAR2(10) | Y | รหัสประเภทใบอนุญาตของ คปภ. เช่น `203`, `204` (ใช้คู่กับ FIN_CODE) |
| 22 | INCOME_CODE | VARCHAR2(20) | Y | รหัสหมวดรายได้สำหรับบัญชี เช่น `OIC002-00003`, `OIC002-00004` |
| 23 | REF_ID | NUMBER(38,0) | Y | FK อ้างอิง ID ของ record ต้นฉบับในตารางเดียวกัน (ใช้กับประเภทที่เป็น "ต่อ" ของประเภทอื่น) |

**ข้อมูลใน Table (IS_ACTIVE = 1 คือ ใช้งานจริง):**

| ID | CODE | DISPLAY_NAME_TH | IS_ACTIVE | FEE_NAME | OIC_CODE | INCOME_CODE |
|:---:|:---:|---|:---:|---|:---:|---|
| 1 | 01 | ตัวแทนประกันชีวิต | 1 | ค่าธรรมเนียมสมัครสอบตัวแทนประกันชีวิต | 201 | OIC002-00001 |
| 2 | 02 | ตัวแทนประกันวินาศภัย | 1 | ค่าธรรมเนียมสมัครสอบตัวแทนประกันวินาศภัย | 202 | OIC002-00002 |
| 3 | 03 | นายหน้าประกันชีวิต | 1 | ค่าธรรมเนียมสมัครสอบนายหน้าประกันชีวิต ประเภทการประกันชีวิตโดยตรง | 203 | OIC002-00003 |
| 4 | 04 | นายหน้าประกันวินาศภัย | 1 | ค่าธรรมเนียมสมัครสอบนายหน้าประกันวินาศภัย ประเภทการประกันวินาศภัยโดยตรง | 204 | OIC002-00004 |
| 5 | 05 | ตัวแทนประกันวินาศภัย เฉพาะกรมธรรม์ประกันภัยอุบัติเหตุส่วนบุคคลและประกันสุขภาพ | 1 | ค่าธรรมเนียมสมัครสอบตัวแทนประกันวินาศภัย ประเภทสุขภาพและอุบัติเหตุ | 202 | OIC002-00002 |
| 6 | 06 | ตัวแทนประกันวินาศภัย เฉพาะกรมธรรม์ประกันภัยตามพระราชบัญญัติคุ้มครองผู้ประสบภัยจากรถ | 1 | ค่าธรรมเนียมสมัครสอบตัวแทนประกันวินาศภัย ประเภท พ.ร.บ. | 202 | OIC002-00002 |

### เลขที่รอบสอบ, ประเภทใบอนุญาต, ประเภทการขึ้นทะเบียน

| เลขที่รอบสอบ	| ประเภทใบอนุญาต |	License name TH |	License name EN |	การเปิดสอบ | FIN_CODE | OIC_CODE |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 6910300001 | 03 | นายหน้าประกันชีวิต | Life Insurance Broker | บุคคล | 203 | 203 |
| 6910400001 | 04 | นายหน้าประกันวินาศภัย | Non Life Insurance Broker | บุคคล | 204 | 204 |
| 6920300001 | 03 | นายหน้าประกันชีวิต | Life Insurance Broker | นิติบุคคล | 203 | 203 |
| 6920400001 | 04 | นายหน้าประกันวินาศภัย | Non Life Insurance Broker | นิติบุคคล | 204 | 204 |
| 6930100001 | 01 | ตัวแทนประกันชีวิต | Life Insurance Agent | หน่วยงานภายนอก | 201 | 201 |
| 6930200001 | 02 | ตัวแทนประกันวินาศภัย | Non Life Insurance Agent | หน่วยงานภายนอก | 202 | 202 |
| 6930300001 | 03 | นายหน้าประกันชีวิต | Life Insurance Broker | หน่วยงานภายนอก | 203 | 203 |
| 6930400001 | 04 | นายหน้าประกันวินาศภัย | Non Life Insurance Broker | หน่วยงานภายนอก | 204 | 204 |
| 6930500001 | 05 | ตัวแทนประกันวินาศภัย เฉพาะกรมธรรม์ประกันภัยอุบัติเหตุส่วนบุคคลและประกันสุขภาพ | Non Life Insurance Agent for Personal Accident Insurance (PA) and Health | หน่วยงานภายนอก | 202 | 202 |
| 6930600001 | 06 | ตัวแทนประกันวินาศภัย เฉพาะกรมธรรม์ประกันภัยตามพระราชบัญญัติคุ้มครองผู้ประสบภัยจากรถ | Non Life Insurance Agent for Compulsory Motor Insurance | หน่วยงานภายนอก | 202 | 202 |
| 6916100001 | 61 (R01) | นายหน้าประกันชีวิต ประเภทการจัดการให้มีการประกันภัยต่อ | Life Reinsurance Broker | บุคคล | 203 | 203 |
| 6916200001 | 62 (R02) | นายหน้าประกันวินาศภัย ประเภทการจัดการให้มีการประกันภัยต่อ | Non Life Reinsurance Broker | บุคคล | 204 | 204 |

---

## Problems

จาก Observation ต้องการ Implement ระบบเพิ่มเติม คือการสอบประเภท "การขึ้นทะเบียน" โดยมี Concept ดังนี้

1. การสอบขึ้นทะเบียน เป็นการสอบเพื่อขึ้นทะเบียน
2. นำมาใช้กับการสอบ คือ การประกันต่อ เบื้องต้น มีการขึ้นทะเบียน 2 ประเภท ดังนี้

|ลำดับ| ประเภท | Type |
|:---:|---|---|
| 1 | นายหน้าประกันชีวิต การประกันต่อ | Life Reinsurance Broker |
| 2 | นายหน้าประกันวินาศภัย การประกันต่อ | Non Life Reinsurance Broker |

3. (Requirement) Project Owner ให้ Concept ว่า ขั้นตอนการเปิดรอบสอบ การสมัครสอบ การชำระเงิน จะเหมือนกับการสอบรอบปกติบุคคลธรรมดา แต่การออกแบบระบบ ให้แยกกับการสอบใบอนุญาต คือ 

   **ตารางประเภท**
   - ประเภทใบอนุญาต จะใช้ตาราง `MT_T_LICENSE_TYPE`
   - ประเภทการสอบขึ้นทะเบียน ให้เพิ่มตาราง `MT_T_REGIST_TYPE` Column ให้คล้ายกับตาราง `MT_T_LICENSE_TYPE`

### Tables MT_T_REGIST_TYPE : ประเภทการขึ้นทะเบียน

| ลำดับ | Column Name | Data Type | Allow Nulls | Field Description |
|---|---|---|:---:|---|
| 1 | ID | NUMBER(10,0) | Y | ID |
| 2 | CODE | NUMBER(10,0) | Y | Primary Key รหัสประเภทการขึ้นทะเบียน |
| 3 | NAME_TH | VARCHAR2(250) | Y | ชื่อประเภทการขึ้นทะเบียนภาษาไทย |
| 4 | NAME_EN | VARCHAR2(250) | Y | ชื่อประเภทการขึ้นทะเบียนภาษาอังกฤษ |
| 5 | ISACTIVE | NUMBER(1,0) | Y | ใช้งาน |
| 6 | CREATE_BY | VARCHAR2(50) | Y | ผู้สร้างข้อมูล |
| 7 | CREATE_DATE | TIMESTAMP(6) | Y | วันที่/เวลาสร้างข้อมูล |
| 8 | UPDATE_BY | VARCHAR2(50) | Y | ผู้แก้ไขข้อมูลล่าสุด |
| 9 | UPDATE_DATE | TIMESTAMP(6) | Y | วันที่/เวลาแก้ไขข้อมูลล่าสุด |
| 10 | FIN_CODE | VARCHAR2(20) | Y | รหัสสำหรับระบบการเงิน |
| 11 | FEE_NAME | VARCHAR2(250) | Y | ชื่อประเภทการขึ้นทะเบียนสำหรับระบบการเงิน |
| 12 | DISPLAY_NAME_TH | VARCHAR2(250) | Y | ชื่อที่แสดงประเภทการขึ้นทะเบียนภาษาไทย |
| 13 | DISPLAY_NAME_EN | VARCHAR2(250) | Y | ชื่อที่แสดงประเภทการขึ้นทะเบียนภาษาอังกฤษ |
| 14 | OIC_CODE | VARCHAR2(20) | Y | รหัสสำหรับระบบการเงิน |
| 15 | INCOME_CODE | VARCHAR2(20) | Y | รหัสสำหรับระบบการเงิน |

#### ข้อมูล Initial MT_T_REGIST_TYPE

| ID | CODE | NAME_TH | NAME_EN | ISACTIVE | CREATE_BY | CREATE_DATE | UPDATE_BY | UPDATE_DATE | FIN_CODE | FEE_NAME | DISPLAY_NAME_TH | DISPLAY_NAME_EN | OIC_CODE | INCOME_CODE |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 61 | นายหน้าประกันชีวิต การประกันต่อ | Life Reinsurance Broker | 1 | 1 | | | | 203 | ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันชีวิต การประกันต่อ | นายหน้าประกันชีวิต การประกันต่อ | Life Reinsurance Broker | 203 | OIC002-00003 |
| 2 | 62 | นายหน้าประกันวินาศภัย การประกันต่อ | Non Life Reinsurance Broker | 1 | 1 | | | | 204 | ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันวินาศภัย การประกันต่อ | นายหน้าประกันวินาศภัย การประกันต่อ | Non Life Reinsurance Broker | 204 | OIC002-00004 |

   **การเลือกใบอนุญาต ตอนที่สร้างรอบสอบ (`/PersonalExamRound_New`)**
   - ที่ ประเภทใบอนุญาต ให้ดึงข้อมูลจาก 2 ตาราง มาแสดงเลย คือ `MT_T_LICENSE_TYPE` และ `MT_T_REGIST_TYPE`
   - เปลี่ยน lable จาก `การเลือกใบอนุญาต` เป็น `การเลือกใบอนุญาต/การขึ้นทะเบียน`

   **การตั้งค่าศูนย์สอบ  (`/ExamCenter/ExamCenter_Detail/50`)**
   - ที่ ประเภทใบอนุญาตที่รับผิดชอบ ให้ดึงข้อมูลจาก 2 ตาราง มาแสดงเลย คือ `MT_T_LICENSE_TYPE` และ `MT_T_REGIST_TYPE`
   - เปลี่ยน lable จาก `ประเภทใบอนุญาตที่รับผิดชอบ` เป็น `ประเภทใบอนุญาต/การขึ้นทะเบียนที่รับผิดชอบ`


3. วิชาสอบ และเกณฑ์คะแนนสอบ <br>
   61 (R01) นายหน้าประกันชีวิต ประเภทการจัดการให้มีการประกันภัยต่อ <br>
   โครงสร้างหลักสูตรการสอบ <br>
   - หมวด 1 ความรู้เกี่ยวกับการประกันภัยต่อ
   - วิชาที่ 1 : วิชาความรู้เกี่ยวกับการประกันภัยต่อ จำนวน 40 ข้อ ข้อละ 2 คะแนน คิดเป็นคะแนนเต็ม 80 คะแนน เกณฑ์การสอบผ่าน 60% (ทำข้อสอบถูกตั้งแต่ 24 ข้อ คิดเป็นคะแนนตั้งแต่ 48 คะแนน)
   
   62 (R02) นายหน้าประกันวินาศภัย ประเภทการจัดการให้มีการประกันภัยต่อ <br>
   โครงสร้างหลักสูตรการสอบ <br>
   - หมวด 1 ความรู้เกี่ยวกับการประกันภัยต่อ
   - วิชาที่ 1 : วิชาความรู้เกี่ยวกับการประกันภัยต่อ จำนวน 40 ข้อ ข้อละ 2 คะแนน คิดเป็นคะแนนเต็ม 80 คะแนน เกณฑ์การสอบผ่าน 60% (ทำข้อสอบถูกตั้งแต่ 24 ข้อ คิดเป็นคะแนนตั้งแต่ 48 คะแนน)

**ยังไม่ได้ข้อมูลรหัสวิชา และชื่อวิชา จาก Project Owner**

---

## Requirement Analysis

### 1. สถานะปัจจุบันของระบบ (As-Is)

จากการสำรวจ Source Code และ Database พบว่า **โครงสร้างหลักของ Feature นี้ถูก Implement ไว้บางส่วนแล้ว** โดยมีสิ่งที่มีอยู่แล้วดังนี้:

#### ✅ สิ่งที่มีอยู่แล้วใน Database

| รายการ | ชื่อ | หมายเหตุ |
|---|---|---|
| Table ประเภทการขึ้นทะเบียน | `MT_T_REGIST_TYPE` | **มีอยู่แล้ว** — structure เหมือน `MT_T_LICENSE_TYPE` ทุก column |
| Field แยกประเภท (Discriminator) | `MT_T_P_EXAM_ROUND.IS_LICENSE_TYPE` | `1` = ใบอนุญาต, `2` = การขึ้นทะเบียน |
| Field FK สำรอง | `MT_T_P_EXAM_ROUND.REGIST_TYPE_ID` | FK ชี้ไปยัง `MT_T_REGIST_TYPE` |
| Field แยกประเภทใน Bill Payment | `IIQE_T_BILL_PAYMENT.IS_LICENSE_TYPE` | มีอยู่แล้ว |
| Field แยกประเภทใน Exam Center | `MT_T_EXAM_CENTER_LICENSE_TYPE.IS_LICENSE_TYPE` | มีอยู่แล้ว ปัจจุบัน = `1` ทั้งหมด |

#### ✅ สิ่งที่มีอยู่แล้วใน Code

| รายการ | ไฟล์/Pattern |
|---|---|
| Repository สำหรับ `MT_T_REGIST_TYPE` | `IIQE_RegistrationTypeRepo.cs`, `IIQE_RegistrationTypeBL.cs` |
| UNION query รวม 2 ตาราง | `ExamCenterRepo.GetLicenseTypeList()` — `SELECT ... FROM MT_T_LICENSE_TYPE UNION ALL SELECT ... FROM MT_T_REGIST_TYPE` |
| CASE statement แยกแสดงชื่อ | `PersonalExamRoundRepo` — `CASE WHEN IS_LICENSE_TYPE = 2 THEN r.DISPLAY_NAME_TH ELSE l.DISPLAY_NAME_TH` |
| ViewModel ที่รองรับ `IS_LICENSE_TYPE` | `IIQE_PersonalExamRound_New_VM.IS_LICENSE_TYPE` |

#### ⚠️ ปัญหาของข้อมูลปัจจุบันใน `MT_T_REGIST_TYPE`

ข้อมูลใน `MT_T_REGIST_TYPE` ที่มีอยู่ **ไม่ตรงกับ Requirement** ที่ต้องการ:

| | DB ปัจจุบัน | Requirement ต้องการ |
|---|---|---|
| ID=1, CODE | `R02` | ควรเป็น `61` |
| ID=1, NAME_TH | `การจัดการประกันชีวิต ประเภทต่อ` | ควรเป็น `นายหน้าประกันชีวิต การประกันต่อ` |
| ID=2, CODE | `R01` | ควรเป็น `62` |
| ID=2, NAME_TH | `การจัดการประกันวินาศภัย ประเภทต่อ` | ควรเป็น `นายหน้าประกันวินาศภัย การประกันต่อ` |
| IS_ACTIVE ทั้งหมด | `NULL` | ควรเป็น `1` |
| OIC_CODE | `NULL` | ควรเป็น `203`, `204` |
| INCOME_CODE | `NULL` | ควรเป็น `OIC002-00003`, `OIC002-00004` (หรือค่าที่ระบบการเงินกำหนด) |

---

### 2. Gap Analysis (สิ่งที่ยังขาด / ต้องทำ)

#### 2.1 Database — Data Gaps

| # | รายการ | Action |
|:---:|---|---|
| 1 | `MT_T_REGIST_TYPE` — ข้อมูลไม่ครบ/ไม่ถูกต้อง (CODE, NAME, IS_ACTIVE, OIC_CODE, INCOME_CODE) | **UPDATE ข้อมูล** |
| 2 | `MT_T_EXAM_CENTER_LICENSE_TYPE` — ยังไม่มี record สำหรับ IS_LICENSE_TYPE=2 (registration type) | **INSERT ข้อมูล** ศูนย์สอบที่รองรับการขึ้นทะเบียน |

#### 2.2 Frontend — UI Gaps

| # | หน้า | รายการที่ต้องแก้ | Action |
|:---:|---|---|---|
| 1 | `/PersonalExamRound/PersonalExamRound_New` | Dropdown `ddlLicenseType` ดึงข้อมูลจาก `GetLicenseType()` — ต้องตรวจสอบว่า method นี้ UNION กับ `MT_T_REGIST_TYPE` แล้วหรือยัง | ตรวจสอบ + แก้ไข |
| 2 | `/PersonalExamRound/PersonalExamRound_New` | Label `ประเภทใบอนุญาต` → `ประเภทใบอนุญาต/การขึ้นทะเบียน` | แก้ label |
| 3 | `/ExamCenter/ExamCenter_Detail` | ส่วนเลือก `ประเภทใบอนุญาตที่รับผิดชอบ` → ต้องแสดง registration type ด้วย | ตรวจสอบ + แก้ไข |
| 4 | `/ExamCenter/ExamCenter_Detail` | Label `ประเภทใบอนุญาตที่รับผิดชอบ` → `ประเภทใบอนุญาต/การขึ้นทะเบียนที่รับผิดชอบ` | แก้ label |

#### 2.3 Business Logic — Logic Gaps

| # | รายการ | Action |
|:---:|---|---|
| 1 | `INSERT MT_T_P_EXAM_ROUND` — ต้องตรวจสอบว่า `IS_LICENSE_TYPE` และ `REGIST_TYPE_ID` ถูก set ถูกต้องเมื่อเลือก registration type | ตรวจสอบ + แก้ไข |
| 2 | Bill Payment — `FEE_NAME` ที่ส่งไป ERP ใน `DESCP_1` ต้องดึงจาก `MT_T_REGIST_TYPE.FEE_NAME` ไม่ใช่ `MT_T_LICENSE_TYPE.FEE_NAME` เมื่อ `IS_LICENSE_TYPE = 2` | ตรวจสอบ + แก้ไข |
| 3 | Exam Schedule (หน้าตารางสอบ) — filter ประเภทใบอนุญาตต้องแสดง registration type ด้วย | ตรวจสอบ |

---

### 3. Impact Analysis

#### High Impact (ต้องแก้แน่นอน)

| ระดับ | Component | รายละเอียด |
|:---:|---|---|
| 🔴 | **Data: `MT_T_REGIST_TYPE`** | Update ข้อมูลให้ถูกต้อง — ทุก flow ขึ้นอยู่กับข้อมูลนี้ |
| 🔴 | **Data: `MT_T_EXAM_CENTER_LICENSE_TYPE`** | Insert ศูนย์สอบที่รองรับ registration type (IS_LICENSE_TYPE=2) — ถ้าไม่มี ระบบ validate ไม่ผ่าน |
| 🔴 | **UI: PersonalExamRound_New** | Dropdown ต้องแสดง registration type — ถ้าไม่มีในตัวเลือก เจ้าหน้าที่เปิดรอบสอบไม่ได้ |
| 🔴 | **BL: INSERT MT_T_P_EXAM_ROUND** | ต้อง set `IS_LICENSE_TYPE=2` และ `REGIST_TYPE_ID` ให้ถูกต้อง |

#### Medium Impact (ตรวจสอบและปรับ)

| ระดับ | Component | รายละเอียด |
|:---:|---|---|
| 🟡 | **UI: ExamCenter_Detail** | Label + dropdown รองรับ registration type |
| 🟡 | **BL: Bill Payment** | FEE_NAME ใน DESCP_1 ต้องแยกดึงจาก MT_T_REGIST_TYPE เมื่อ IS_LICENSE_TYPE=2 |
| 🟡 | **UI: ExamSchedule** | หน้าตารางสอบต้องแสดง registration type ใน filter และ calendar |

#### Low Impact (น่าจะรองรับอยู่แล้ว)

| ระดับ | Component | รายละเอียด |
|:---:|---|---|
| 🟢 | **BL: ExamCenterRepo** | UNION query มีอยู่แล้ว — ตรวจสอบ IS_ACTIVE |
| 🟢 | **BL: PersonalExamRoundRepo CASE** | แยก DISPLAY_NAME_TH ตาม IS_LICENSE_TYPE มีอยู่แล้ว |
| 🟢 | **IIQE_T_BILL_PAYMENT** | IS_LICENSE_TYPE field มีอยู่แล้ว |
| 🟢 | **ViewModel** | IS_LICENSE_TYPE ใน PersonalExamRound_New_VM มีอยู่แล้ว |

---

### 4. การส่งข้อมูลรอบสอบไประบบ Exam (EXAM System)

**ที่มา:** ระบบมี Batch Process ชื่อ `IIQE_SEND_P_EXAM_ROUND` ที่ส่งข้อมูลรอบสอบไปยังระบบ EXAM ผ่าน API

#### สถานะของ Code

**✅ Code รองรับ IS_LICENSE_TYPE = 2 แล้ว** (ไฟล์: `API_BATCH_m/API_BATCH/Managers/IIQE_SEND_P_EXAM_ROUND.cs`)

```csharp
if (dr["IS_LICENSE_TYPE"].ToString() == "1")
{
    scd_data.EXAM_LICENSE = dr["EXAM_LICENSE"].ToString(); // ส่ง LICENSE CODE ไป EXAM System
}
if (dr["IS_LICENSE_TYPE"].ToString() == "2")
{
    string registypeid = _sendExamRound.MT_T_REGIST_TYPE(dr["REGIST_TYPE_ID"].ToString());
    scd_data.REGIST_TYPE_ID = Convert.ToInt32(registypeid); // ส่ง REGIST_TYPE_ID ไป EXAM System
}
```

Query ดึงข้อมูลรอบสอบก็ SELECT ทั้ง `IS_LICENSE_TYPE` และ `REGIST_TYPE_ID` มาแล้ว

#### ⚠️ ปัญหาที่พบ: REGIST_TYPE_ID ใน MT_T_P_EXAM_ROUND

Code ตรวจสอบ `REGIST_TYPE_ID` โดย query ไปที่ `MT_T_REGIST_TYPE WHERE ID = :ID` — ถ้า `REGIST_TYPE_ID` ใน `MT_T_P_EXAM_ROUND` เป็น NULL หรือ ID ไม่ตรงกับ `MT_T_REGIST_TYPE` จะส่งข้อมูลผิดพลาด

> **สาเหตุ:** `MT_T_REGIST_TYPE` ปัจจุบัน ID=1 (CODE=R02), ID=2 (CODE=R01) และข้อมูล Requirement ต้องการ ID=1 (CODE=61), ID=2 (CODE=62) — ต้อง UPDATE ข้อมูลให้ถูกต้องก่อน

#### Payload ที่ส่งไป EXAM System

| Field | ที่มา | กรณี IS_LICENSE_TYPE=1 | กรณี IS_LICENSE_TYPE=2 |
|---|---|---|---|
| `EXAM_LICENSE` | `MT_T_LICENSE_TYPE.CODE` | ✅ ส่งรหัสใบอนุญาต | ไม่ส่ง |
| `REGIST_TYPE_ID` | `MT_T_REGIST_TYPE.ID` | ไม่ส่ง | ✅ ส่ง ID ของประเภทขึ้นทะเบียน |
| `TESTING_NO` | `MT_T_P_EXAM_ROUND.CODE` | ✅ | ✅ |
| `EXAM_DATE` | `MT_T_P_EXAM_ROUND.EXAM_DATE` | ✅ | ✅ |
| `EXAM_PV` | `MT_T_PROVINCE.CODE` | ✅ | ✅ |
| `EXAM_P` | `MT_T_EXAM_LOCATION.AG_CODE` (รหัสสถานที่สอบในระบบ EXAM) | ✅ | ✅ |

#### สรุป Flow 1

| | รายการ | สถานะ |
|:---:|---|:---:|
| ✅ | Code logic แยก IS_LICENSE_TYPE 1/2 มีอยู่แล้ว | ไม่ต้องแก้ code |
| 🔴 | ข้อมูลใน MT_T_REGIST_TYPE ต้อง UPDATE ให้ถูกต้องก่อน | **ต้องทำ** |
| 🔴 | ตอนสร้างรอบสอบ (INSERT MT_T_P_EXAM_ROUND) ต้อง set REGIST_TYPE_ID ให้ถูกต้อง | **ต้องตรวจสอบ** |

---

### 5. การรับผลสอบ (Exam Results)

หลังจากระบบ EXAM ประมวลผลผลสอบ จะส่งข้อมูลกลับมายังระบบ IIQE เพื่อเก็บและอนุมัติ

#### ตาราง Master ที่เกี่ยวข้อง

**`MT_T_EXAM_SCORE_SETTING` — เกณฑ์คะแนนสอบ**

| Column | Description |
|---|---|
| `LICENSE_TYPE_ID` | FK ชี้ไปยัง MT_T_LICENSE_TYPE หรือ MT_T_REGIST_TYPE (ขึ้นกับ IS_LICENSE_TYPE) |
| `IS_LICENSE_TYPE` | `1` = ใบอนุญาต, `2` = ขึ้นทะเบียน |
| `START_DATE`, `END_DATE` | ช่วงวันที่ที่เกณฑ์นี้มีผล |
| `IS_ACTIVE` | สถานะ |

> JOIN กับ `MT_T_EXAM_SCORE_SETTING_DT` เพื่อดูรายวิชา + PERCENT_PASS + MAX_SCORE

**`MT_T_SUBJECT_ITEM` — รายวิชาสอบ**

| Column | Description |
|---|---|
| `LICENSE_TYPE_ID` | FK ชี้ไปยัง MT_T_LICENSE_TYPE หรือ MT_T_REGIST_TYPE |
| `IS_LICENSE_TYPE` | `1` = ใบอนุญาต, `2` = ขึ้นทะเบียน |
| `SUBJECT_GROUP_ID` | FK → `MT_T_SUBJECT_GROUP` (กลุ่มวิชา) |
| `CODE`, `NAME_TH` | รหัสและชื่อวิชา |
| `MAX_SCORE` | คะแนนเต็ม |
| `AG_CODE` | รหัสวิชาในระบบ AG |
| `TRANSFER_STATUS` | สถานะส่งข้อมูลไประบบ AG (`0`=รอ, `1`=ส่งแล้ว) |

#### ⚠️ ปัญหาที่พบ: ข้อมูลขาดสำหรับ Registration Type

จากการ Query DB จริง พบว่า:

**`MT_T_EXAM_SCORE_SETTING`** — **ไม่มีข้อมูลสำหรับ IS_LICENSE_TYPE = 2 เลย**
- มีข้อมูล IS_ACTIVE=1 เฉพาะ LICENSE_TYPE_ID 1-6 (IS_LICENSE_TYPE=1 ทั้งหมด)
- ไม่มี record ที่ IS_LICENSE_TYPE=2

**`MT_T_SUBJECT_ITEM`** — **มีข้อมูล IS_LICENSE_TYPE=2 เพียง 1 วิชา (ไม่สมบูรณ์)**
- ID=37, LICENSE_TYPE_ID=17 (ชี้ไปยัง MT_T_LICENSE_TYPE ที่ IS_ACTIVE=0 — ไม่ถูกต้อง)
- ยังไม่มีวิชาที่ผูกกับ `MT_T_REGIST_TYPE` ID=1 (61) หรือ ID=2 (62)
- **ไม่มีเกณฑ์คะแนน** (MT_T_EXAM_SCORE_SETTING_DT) สำหรับ registration type เลย

#### โครงสร้างหลักสูตรการสอบ (จาก Project Owner)

**ประเภท 61 — นายหน้าประกันชีวิต ประเภทการจัดการให้มีการประกันภัยต่อ** <br>
**ประเภท 62 — นายหน้าประกันวินาศภัย ประเภทการจัดการให้มีการประกันภัยต่อ**

| หมวด | วิชา | จำนวนข้อ | คะแนนเต็ม | เกณฑ์ผ่าน |
|---|---|:---:|:---:|---|
| หมวด 1: ความรู้เกี่ยวกับการประกันภัยต่อ | วิชาความรู้เกี่ยวกับการประกันภัยต่อ | 40 ข้อ × 2 คะแนน | 80 | 60% (≥48 คะแนน / ≥24 ข้อ) |

> **ข้อสังเกต:** โครงสร้างหลักสูตร 61 และ 62 **เหมือนกันทุกประการ** — หมวดเดียว วิชาเดียว คะแนนเดียวกัน เกณฑ์ผ่านเดียวกัน

#### วิเคราะห์: รหัสวิชา / ชื่อวิชา และการแยก Record

**คำถาม 1 — `AG_CODE` (รหัสวิชาในระบบ EXAM) จำเป็นหรือไม่?**

**ใช่ — จำเป็นมาก** `AG_CODE` คือรหัสที่ระบบ EXAM ใช้ระบุวิชาเมื่อรับผลสอบกลับมา ถ้าไม่มีหรือผิด ระบบ IIQE จะ match ผลสอบไม่ได้

- `CODE` และ `NAME_TH` ใน `MT_T_SUBJECT_ITEM` เป็น internal IIQE → **generate เองได้** (running ต่อจาก 00037)
- `AG_CODE` → **ต้องได้รับจาก Project Owner** เพราะระบบ EXAM กำหนด

**คำถาม 2 — 61 และ 62 ควรแยก record หรือใช้ร่วมกัน?**

**ต้องแยก** — เพราะ `MT_T_SUBJECT_ITEM.LICENSE_TYPE_ID` ผูกกับ `MT_T_REGIST_TYPE` แบบ 1-to-many และระบบ filter ด้วย `LICENSE_TYPE_ID + IS_LICENSE_TYPE` คู่กันเสมอ ดูตัวอย่างจากข้อมูลที่มีอยู่:

| ตัวอย่าง | LICENSE_TYPE_ID | AG_CODE | หมายเหตุ |
|---|:---:|:---:|---|
| จรรยาบรรณนายหน้าชีวิต (ID=9) | 3 | `001` | แยก record |
| จรรยาบรรณนายหน้าวินาศ (ID=13) | 4 | `001` | แยก record — AG_CODE ซ้ำได้ในคนละ LICENSE_TYPE |

> ดังนั้น แม้วิชา 61 และ 62 จะมีชื่อเหมือนกัน ต้องสร้าง **2 record แยกกัน** ใน `MT_T_SUBJECT_ITEM` (LICENSE_TYPE_ID=1 สำหรับ 61 และ LICENSE_TYPE_ID=2 สำหรับ 62) และ **AG_CODE อาจเหมือนหรือต่างกันก็ได้** ขึ้นอยู่กับที่ระบบ EXAM กำหนด

#### สิ่งที่ต้องเพิ่มในฐานข้อมูล

| ตาราง | Action | รายละเอียด |
|---|---|---|
| `MT_T_SUBJECT_GROUP` | ตรวจสอบ | `ความรู้เกี่ยวกับการประกันภัยต่อ` (ID=3) มีอยู่แล้ว ✅ |
| `MT_T_SUBJECT_ITEM` | **INSERT 2 records** | วิชาประกันภัยต่อสำหรับ 61 (LICENSE_TYPE_ID=1, IS_LICENSE_TYPE=2) และ 62 (LICENSE_TYPE_ID=2, IS_LICENSE_TYPE=2) แยกกัน |
| `MT_T_EXAM_SCORE_SETTING` | **INSERT 2 records** | เกณฑ์คะแนนสำหรับ LICENSE_TYPE_ID=1 และ LICENSE_TYPE_ID=2 (IS_LICENSE_TYPE=2, IS_ACTIVE=1) |
| `MT_T_EXAM_SCORE_SETTING_DT` | **INSERT 2 records** | PERCENT_PASS=60, MAX_SCORE=80 เชื่อมกับ SUBJECT_ITEM ของ 61 และ 62 ตามลำดับ |

> 🔴 **Blocked:** ต้องได้รับ `AG_CODE` สำหรับวิชานี้จาก Project Owner ก่อน จึงจะ Insert `MT_T_SUBJECT_ITEM` ได้

#### Code รองรับแล้ว — ไม่ต้องแก้

`MT_T_EXAM_SCORE_SETTING` query ใช้ CASE เลือก join table ตาม IS_LICENSE_TYPE อยู่แล้ว:
```sql
CASE WHEN exam.is_license_type = 2 
     THEN (r.code || ' ' || r.display_name_th)  -- MT_T_REGIST_TYPE
     ELSE (l.code || ' ' || l.display_name_th)   -- MT_T_LICENSE_TYPE
END as NAME_TH
```

`MT_T_SUBJECT_ITEM` query filter ด้วยทั้ง `LICENSE_TYPE_ID` AND `IS_LICENSE_TYPE` อยู่แล้ว

#### สรุป Flow 2

| | รายการ | สถานะ |
|:---:|---|:---:|
| ✅ | Code logic ใน Scoring Setting รองรับ IS_LICENSE_TYPE=2 แล้ว | ไม่ต้องแก้ code |
| ✅ | Code logic ใน Subject Item รองรับ IS_LICENSE_TYPE=2 แล้ว | ไม่ต้องแก้ code |
| 🔴 | ไม่มีข้อมูลเกณฑ์คะแนนสอบ (MT_T_EXAM_SCORE_SETTING) สำหรับ registration type | **ต้องเพิ่มข้อมูล** |
| 🔴 | รายวิชาสอบ (MT_T_SUBJECT_ITEM) สำหรับ registration type ยังไม่ครบ | **ต้องเพิ่มข้อมูล** |
| 🟡 | ต้องได้รับข้อมูลรายวิชาและเกณฑ์คะแนนจาก Project Owner / ฝ่ายวิชาการ | **ต้องถาม** |

---

### 6. สรุป Action Items (ทั้งหมด)

```
[Database Scripts — ทำได้ทันที]
1. UPDATE MT_T_REGIST_TYPE — แก้ข้อมูลให้ถูกต้อง (CODE, NAME, IS_ACTIVE, OIC_CODE, INCOME_CODE)
   OIC_CODE: 61=203, 62=204 / INCOME_CODE: 61=OIC002-00003, 62=OIC002-00004 ✅
2. INSERT MT_T_EXAM_CENTER_LICENSE_TYPE — เพิ่ม EXAM_CENTER_ID=7 (สำนักงานใหญ่ รัชดา)
   สำหรับ LICENSE_TYPE_ID=1 (61) IS_LICENSE_TYPE=2 และ LICENSE_TYPE_ID=2 (62) IS_LICENSE_TYPE=2
4. INSERT MT_T_EXAM_SCORE_SETTING (IS_LICENSE_TYPE=2, START_DATE=01/01/2026, END_DATE=01/01/2032)
   + MT_T_EXAM_SCORE_SETTING_DT (PERCENT_PASS=60, MAX_SCORE=80) — รอ SUBJECT_ITEM_ID จากข้อ 3

[Database Scripts — ต้องรอข้อมูลจาก Project Owner]
3. INSERT MT_T_SUBJECT_ITEM — รอ AG_CODE และชื่อวิชาอย่างเป็นทางการ (ข้อ 1, 2)

[Code Changes — UI]
5. ตรวจสอบ GetLicenseType() ใน PersonalExamRoundBL — UNION กับ MT_T_REGIST_TYPE แล้วหรือยัง
6. แก้ Label บนหน้า PersonalExamRound_New.cshtml
7. แก้ Label บนหน้า ExamCenter_Detail.cshtml

[Code Changes — BL]
8. ตรวจสอบ INSERT MT_T_P_EXAM_ROUND — set IS_LICENSE_TYPE=2 + REGIST_TYPE_ID ถูกต้อง
9. ตรวจสอบ Bill Payment FEE_NAME logic — ดึงจาก MT_T_REGIST_TYPE เมื่อ IS_LICENSE_TYPE=2
10. ตรวจสอบหน้า ExamSchedule filter — แสดง registration type ใน dropdown
```

---

## ข้อมูลที่ต้องสอบถามเพิ่มเติมจาก Project Owner

| # | หัวข้อ | รายละเอียด | ผลกระทบถ้าไม่ได้ |
|:---:|---|---|---|
| 1 | **AG_CODE ของวิชาสอบ** | รหัสวิชา "ความรู้เกี่ยวกับการประกันภัยต่อ" สำหรับประเภท 61 และ 62 ในระบบ EXAM คืออะไร? (ปัจจุบัน ID=37 ใช้ AG_CODE=`015` แต่ผูกกับ LICENSE_TYPE ที่ IS_ACTIVE=0) | ❌ Block การ INSERT `MT_T_SUBJECT_ITEM` — ระบบ EXAM จะ match ผลสอบไม่ได้ |
| 2 | **ชื่อวิชา (NAME_TH) อย่างเป็นทางการ** | ชื่อเต็มอย่างเป็นทางการของวิชา "ความรู้เกี่ยวกับการประกันภัยต่อ" สำหรับ 61 และ 62 เหมือนกันหรือต่างกัน? | ❌ Block การ INSERT `MT_T_SUBJECT_ITEM` |
| 3 | ~~**ศูนย์สอบที่รองรับการขึ้นทะเบียน**~~ | ✅ **ตอบแล้ว:** เฉพาะ **สำนักงาน คปภ. (สำนักงานใหญ่)** — `EXAM_CENTER_ID = 7`, CODE = `I0001` ปัจจุบันมี IS_LICENSE_TYPE=1 สำหรับ LICENSE_TYPE_ID 3, 4 อยู่แล้ว (ID=13, 14) ต้อง INSERT เพิ่มสำหรับ IS_LICENSE_TYPE=2 | — |
| 4 | ~~**ข้อมูล OIC_CODE และ INCOME_CODE**~~ | ✅ **ตอบแล้ว:** 61 (ชีวิต) = OIC_CODE `203`, INCOME_CODE `OIC002-00003` / 62 (วินาศ) = OIC_CODE `204`, INCOME_CODE `OIC002-00004` | — |
| 5 | ~~**ค่าธรรมเนียมสอบ**~~ | ✅ **ตอบแล้ว:** **200 บาท** เหมือนนายหน้าปกติ | — |
| 6 | ~~**START_DATE / END_DATE ของเกณฑ์คะแนน**~~ | ✅ **ตอบแล้ว:** START_DATE = `01/01/2026`, END_DATE = `01/01/2032` | — |
| 9 | ~~**FEE_NAME ในใบแจ้งชำระเงิน**~~ | ✅ **ตอบแล้ว:** ระบุไว้ใน Initial data ของ `MT_T_REGIST_TYPE` แล้ว: 61 = `ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันชีวิต การประกันต่อ`, 62 = `ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันวินาศภัย การประกันต่อ` | — |
| 10 | **กระบวนการหลังสอบผ่าน** | หลัง Approve ผลสอบแล้ว กระบวนการถัดไปสำหรับประเภทขึ้นทะเบียนคืออะไร? มีการส่งข้อมูลหรือออกเอกสารเพิ่มเติมหรือไม่? | ⚠️ กระทบ scope งาน — อาจต้องพัฒนา flow เพิ่ม |
| 11 | **การแสดงผลบน Dashboard** | คำขอสมัครสอบขึ้นทะเบียน (61/62) จะแสดงรวมใน widget "คำขอสมัครสอบรอบปกติ" หรือเพิ่ม widget / section ใหม่แยก? | ⚠️ กระทบ UI Dashboard และ scope งาน |

## ข้อมูลที่ต้องสอบถามเพิ่มเติมจากทีม EXAM System

| # | หัวข้อ | รายละเอียด | ผลกระทบถ้าไม่ได้ |
|:---:|---|---|---|
| 7 | **REGIST_TYPE_ID ที่ระบบ EXAM รับรู้** | ระบบ EXAM รู้จัก `REGIST_TYPE_ID` = 1 (61) และ = 2 (62) แล้วหรือยัง? หรือต้องแจ้งให้ทีม EXAM เพิ่มข้อมูลก่อน? | ❌ Block การส่งรอบสอบไประบบ EXAM — `IIQE_SEND_P_EXAM_ROUND` validate ว่า REGIST_TYPE_ID มีอยู่ในระบบ EXAM |
| 8 | **รูปแบบ payload ที่ EXAM รับ** | field `REGIST_TYPE_ID` ใน payload ปัจจุบัน (SendExamRoundVM) ถูกต้องและระบบ EXAM พร้อมรับแล้วหรือยัง? | ❌ Block การส่งรอบสอบ |

## Code Bugs ที่พบจากการวิเคราะห์ (ต้องแก้ไข)

> พบ bug ที่จะทำให้ระบบทำงานผิดพลาดเมื่อสร้างรอบสอบประเภทขึ้นทะเบียน ไม่ต้องถาม Project Owner — แก้ได้เลย

| # | ไฟล์ | Method | ปัญหา | ผลกระทบ |
|:---:|---|---|---|---|
| B1 | `IIQE_PersonalExamRoundRepo.cs` | `getLicenseCodeByLicenseTypeID()` | Query เฉพาะ `MT_T_LICENSE_TYPE` — เมื่อ IS_LICENSE_TYPE=2 ส่ง LICENSE_TYPE_ID=1 จะได้ CODE=`01` แทน `61` | ❌ รหัสรอบสอบ (CODE) ผิด เช่น ได้ `6910100001` แทน `6916100001` |
| B2 | `IIQE_PersonalExamRoundRepo.cs` | `getRunningByLicenseTypeID()` | ใช้ `RUNNING_LICENSETYPE` เป็น key โดยไม่แยก IS_LICENSE_TYPE — REGIST_TYPE ID=1 ชนกับ LICENSE_TYPE ID=1 | ❌ Running number รอบขึ้นทะเบียนนับต่อจากรอบตัวแทนประกันชีวิต |
| B3 | `IIQE_ApproveExamResultRepo.cs` | `getLicenseTypeForSearch()` | Query เฉพาะ `MT_T_LICENSE_TYPE` ไม่มี UNION กับ `MT_T_REGIST_TYPE` | ❌ Filter ในหน้าอนุมัติผลสอบไม่แสดง 61 และ 62 |

---

## Task

1. วางแผนการ Implement ระบบ เพื่อให้รองรับการเปิดรอบสอบ (ใช้ Plan mode)
2. ตรวจสอบผลกระทบ
3. หากต้องการทดสอบการประมวลผลสอบ สามารถทดสอบได้ไหม มีแผนอย่างไรบ้าง จะต้องเอาข้อมูลมาใส่ตาราง TEMP.... อย่างไรบ้าง
4. หากมี Script Database ที่จะต้องไป Execute ให้เตรียมให้ด้วย

---

## Database Scripts

### Script 1: UPDATE MT_T_REGIST_TYPE (พร้อมรัน ✅)

```sql
-- แก้ข้อมูล MT_T_REGIST_TYPE ให้ถูกต้องครบถ้วน
UPDATE MT_T_REGIST_TYPE
SET
    CODE             = '61',
    NAME_TH          = 'นายหน้าประกันชีวิต การประกันต่อ',
    NAME_EN          = 'Life Reinsurance Broker',
    IS_ACTIVE        = 1,
    DISPLAY_NAME_TH  = 'นายหน้าประกันชีวิต การประกันต่อ',
    DISPLAY_NAME_EN  = 'Life Reinsurance Broker',
    FIN_CODE         = '203',
    OIC_CODE         = '203',
    INCOME_CODE      = 'OIC002-00003',
    FEE_NAME         = 'ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันชีวิต การประกันต่อ',
    UPDATE_BY        = 1,
    UPDATE_DATE      = SYSDATE
WHERE ID = 1;

UPDATE MT_T_REGIST_TYPE
SET
    CODE             = '62',
    NAME_TH          = 'นายหน้าประกันวินาศภัย การประกันต่อ',
    NAME_EN          = 'Non-Life Reinsurance Broker',
    IS_ACTIVE        = 1,
    DISPLAY_NAME_TH  = 'นายหน้าประกันวินาศภัย การประกันต่อ',
    DISPLAY_NAME_EN  = 'Non-Life Reinsurance Broker',
    FIN_CODE         = '204',
    OIC_CODE         = '204',
    INCOME_CODE      = 'OIC002-00004',
    FEE_NAME         = 'ค่าธรรมเนียมสมัครสอบขึ้นทะเบียนนายหน้าประกันวินาศภัย การประกันต่อ',
    UPDATE_BY        = 1,
    UPDATE_DATE      = SYSDATE
WHERE ID = 2;

COMMIT;
```

---

### Script 2: INSERT MT_T_EXAM_CENTER_LICENSE_TYPE (พร้อมรัน ✅)

> เพิ่ม EXAM_CENTER_ID=7 (สำนักงาน คปภ. สำนักงานใหญ่ รัชดา) สำหรับประเภทขึ้นทะเบียน

```sql
-- ตรวจสอบ MAX(ID) ก่อน INSERT
-- SELECT MAX(ID) FROM MT_T_EXAM_CENTER_LICENSE_TYPE;

INSERT INTO MT_T_EXAM_CENTER_LICENSE_TYPE (ID, EXAM_CENTER_ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, CREATE_BY, CREATE_DATE)
VALUES (SEQ_MT_T_EXAM_CENTER_LICENSE_TYPE.NEXTVAL, 7, 1, 2, 1, SYSDATE);  -- 61 นายหน้าชีวิต ต่อ

INSERT INTO MT_T_EXAM_CENTER_LICENSE_TYPE (ID, EXAM_CENTER_ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, CREATE_BY, CREATE_DATE)
VALUES (SEQ_MT_T_EXAM_CENTER_LICENSE_TYPE.NEXTVAL, 7, 2, 2, 1, SYSDATE);  -- 62 นายหน้าวินาศ ต่อ

COMMIT;
```

> **หมายเหตุ:** ตรวจสอบชื่อ SEQUENCE ก่อนรัน — หากไม่มี SEQUENCE ให้ใช้ `(SELECT NVL(MAX(ID),0)+1 FROM MT_T_EXAM_CENTER_LICENSE_TYPE)` แทน

---

### Script 3: INSERT MT_T_SUBJECT_ITEM (รอ AG_CODE และชื่อวิชา ❌ Block)

```sql
-- TODO: รอข้อมูลจาก Project Owner
-- AG_CODE สำหรับ 61 = ???
-- AG_CODE สำหรับ 62 = ???
-- NAME_TH อย่างเป็นทางการ = ???

-- INSERT MT_T_SUBJECT_ITEM (LICENSE_TYPE=61)
INSERT INTO MT_T_SUBJECT_ITEM (ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, SUBJECT_GROUP_ID, AG_CODE, NAME_TH, IS_ACTIVE, CREATE_BY, CREATE_DATE)
VALUES (SEQ_MT_T_SUBJECT_ITEM.NEXTVAL, 1, 2, 3, '???_61', '???', 1, 1, SYSDATE);

-- INSERT MT_T_SUBJECT_ITEM (LICENSE_TYPE=62)
INSERT INTO MT_T_SUBJECT_ITEM (ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, SUBJECT_GROUP_ID, AG_CODE, NAME_TH, IS_ACTIVE, CREATE_BY, CREATE_DATE)
VALUES (SEQ_MT_T_SUBJECT_ITEM.NEXTVAL, 2, 2, 3, '???_62', '???', 1, 1, SYSDATE);

COMMIT;
```

---

### Script 4: INSERT MT_T_EXAM_SCORE_SETTING + MT_T_EXAM_SCORE_SETTING_DT (รอ SUBJECT_ITEM_ID ❌ Block)

```sql
-- TODO: รอ ID ของ MT_T_SUBJECT_ITEM จาก Script 3

-- === 61 นายหน้าประกันชีวิต การประกันต่อ ===
-- วิชาเดียว: ความรู้เกี่ยวกับการประกันภัยต่อ (MAX_SCORE=80, PERCENT_PASS=60%)
DECLARE
    v_score_id  NUMBER;
    v_item_id_61 NUMBER;
BEGIN
    SELECT ID INTO v_item_id_61 FROM MT_T_SUBJECT_ITEM WHERE IS_LICENSE_TYPE=2 AND LICENSE_TYPE_ID=1 AND ROWNUM=1;

    INSERT INTO MT_T_EXAM_SCORE_SETTING (ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, START_DATE, END_DATE, CREATE_BY, CREATE_DATE)
    VALUES (SEQ_MT_T_EXAM_SCORE_SETTING.NEXTVAL, 1, 2, TO_DATE('01/01/2026','DD/MM/YYYY'), TO_DATE('01/01/2032','DD/MM/YYYY'), 1, SYSDATE)
    RETURNING ID INTO v_score_id;

    INSERT INTO MT_T_EXAM_SCORE_SETTING_DT (ID, EXAM_SCORE_SETTING_ID, SUBJECT_ITEM_ID, MAX_SCORE, PERCENT_PASS, CREATE_BY, CREATE_DATE)
    VALUES (SEQ_MT_T_EXAM_SCORE_SETTING_DT.NEXTVAL, v_score_id, v_item_id_61, 80, 60, 1, SYSDATE);
END;
/

-- === 62 นายหน้าประกันวินาศภัย การประกันต่อ ===
DECLARE
    v_score_id  NUMBER;
    v_item_id_62 NUMBER;
BEGIN
    SELECT ID INTO v_item_id_62 FROM MT_T_SUBJECT_ITEM WHERE IS_LICENSE_TYPE=2 AND LICENSE_TYPE_ID=2 AND ROWNUM=1;

    INSERT INTO MT_T_EXAM_SCORE_SETTING (ID, LICENSE_TYPE_ID, IS_LICENSE_TYPE, START_DATE, END_DATE, CREATE_BY, CREATE_DATE)
    VALUES (SEQ_MT_T_EXAM_SCORE_SETTING.NEXTVAL, 2, 2, TO_DATE('01/01/2026','DD/MM/YYYY'), TO_DATE('01/01/2032','DD/MM/YYYY'), 1, SYSDATE)
    RETURNING ID INTO v_score_id;

    INSERT INTO MT_T_EXAM_SCORE_SETTING_DT (ID, EXAM_SCORE_SETTING_ID, SUBJECT_ITEM_ID, MAX_SCORE, PERCENT_PASS, CREATE_BY, CREATE_DATE)
    VALUES (SEQ_MT_T_EXAM_SCORE_SETTING_DT.NEXTVAL, v_score_id, v_item_id_62, 80, 60, 1, SYSDATE);
END;
/

COMMIT;
```
