## Prompt 

## BILLPAYMENT UPDATE ไม่ถูก DOWNLOAD_ATTACHMENT_ID ที่ IIQE_T_BILL_PAYMENT
### Prompt 2.1
```
Context:
Database: Oracle

Schema files:
- Brain_IIQE\iiqe_database\oic_iiqe_database_schema.md
- Brain_IIQE\iiqe_database\oic_iiqe_database_ai_schema.csv
- Brain_IIQE\iiqe_database\oic_iiqe_primary_key.csv
- Brain_IIQE\iiqe_database\oic_iiqe_relationship_candidates.csv

Step:
การสมัครสอบรอบสอบปกติบุคคลธรรมดา มีขั้นตอนคือ
1. ผู้สมัครสมัครสอบ เลือกรอบสอบ และกดสมัครสอบ
สร้าง รายการที่ IIQE_T_P_EXAM_REQ
2. กดปุ่มชำระเงิน สร้างรายการที IIQE_T_BILL_PAYMENT
3. ส่งข้อมูลการสร้างใบแจ้งการชำระเงิน ไปยังระบบภายนอก (ระบบ ERP ผ่าน Web API)
4. ระบบภายนอกส่งข้อมูลใบแจ้งการชำระเงิน กลับมายัง IIQE
5. สร้างรายการในตาราง MT_T_DOWNLOAD_ATTACHMENT
6. ระบบ UPDATE ข้อมูล ID ของใบแจ้งการชำระเงิน จากตาราง MT_T_DOWNLOAD_ATTACHMENT ไปยัง IIQE_T_BILL_PAYMENT
ที่ Column DOWNLOAD_ATTACHMENT_ID
7. ผู้สมัคร ตรวจสอบใบแจ้งการชำระเงิน และชำระเงิน
8. ระบบภายนอกส่งข้อมูลการชำระเงิน กลับมายัง IIQE และ UPDATE ข้อมูลการชำระเงิน ที่ IIQE_T_RECEIPT

Problem: เรื่อง BILLPAYMENT UPDATE ไม่ถูก DOWNLOAD_ATTACHMENT_ID ที่ IIQE_T_BILL_PAYMENT
1. รายการ IIQE_T_P_EXAM_REQ 1:1 กับ IIQE_T_BILL_PAYMENT
2. แต่ว่า ID ของ MT_T_DOWNLOAD_ATTACHMENT มา UPDATE ที่ IIQE_T_BILL_PAYMENT มากกว่า 1 รายการ
3. ทำให้ผู้สมัครสอบได้รับ BILLPAYMENT ไม่ใช่ของตัวเอง แล้วไปชำระเงิน และ BILLPAYMENT ของตัวเองไม่ถูกชำระเงิน
4. ทำให้ผู้สมัครสอบไม่มีรายชื่อเข้าสอบ และสถานะเป็นไม่ชำระเงินภายในเวลาที่กำหนด
ตรวจสอบจาก query นี้ได้
SELECT
    ID, P_EXAM_REQ_ID, C_EXAM_REQ_ID, COMPANY_CODE, BILLER_ID,
    PAY_DATE, DUE_DATE, REF_NO_1, REF_NO_2, REF_CODE, AMOUNT,
    QR_CODE, STATUS, TRANFER_DATE, TRANFER_STATUS, REMARK,
    CREATE_BY, CREATE_DATE, UPDATE_BY, UPDATE_DATE,
    RUNNING_NUMBER, RUNNING_MONTH, MEMBER_TYPE,
    C_EXAM_REQ_ROUND_ID, DOWNLOAD_ATTACHMENT_ID,
    G_EXAM_REQ_ROUND_ID, G_EXAM_REQ_ID, LICENSE_TYPE_ID,
    C_EXAM_REQ_LICENSE_TYPE_ID, IS_LICENSE_TYPE,
    PERSONAL_MEMBER_ID, G_EXAM_REQ_LICENSE_TYPE_ID
FROM OICIIQE.IIQE_T_BILL_PAYMENT T
WHERE T.DOWNLOAD_ATTACHMENT_ID IN (
    SELECT DOWNLOAD_ATTACHMENT_ID
    FROM OICIIQE.IIQE_T_BILL_PAYMENT
    WHERE DOWNLOAD_ATTACHMENT_ID IS NOT NULL AND P_EXAM_REQ_ID IS NOT NULL
      AND CREATE_DATE BETWEEN TO_DATE('2026-01-01', 'YYYY-MM-DD')
                          AND TO_DATE('2026-01-31', 'YYYY-MM-DD')
    GROUP BY DOWNLOAD_ATTACHMENT_ID
    HAVING COUNT(*) > 1           -- กลุ่มที่มีอย่างน้อย 2 แถวในช่วงวันที่นี้
)
AND T.CREATE_DATE BETWEEN TO_DATE('2026-01-01', 'YYYY-MM-DD')
                      AND TO_DATE('2026-01-31', 'YYYY-MM-DD')
ORDER BY T.CREATE_DATE DESC, T.DOWNLOAD_ATTACHMENT_ID, T.ID;

ConnectionString:
"DefaultConnection": "DATA SOURCE=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.138)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB)));User Id=oiciiqe;Password=oiciiqe;"

Task:
1. วิเคราะห์ว่าปัญหานี้เกิดจากอะไร GET MAX ID หรือไม่
2. แนวทางการแก้ไขปัญหา หรือต้องเพิ่ม Column
```
