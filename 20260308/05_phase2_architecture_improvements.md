# 🏗️ คู่มือแก้ไข Phase 2: Architecture Improvements

**ระยะเวลา:** สัปดาห์ที่ 5-12  
**เป้าหมาย:** ปรับโครงสร้างพื้นฐานให้พร้อมสำหรับ refactoring  
**Prerequisite:** Phase 1 เสร็จสมบูรณ์

---

## Task 2.1: สร้าง Shared Utility Library

### ปัญหา
Utility methods เช่น `ConvertToInt64`, `ConvertToList<T>`, `CheckCurrentCulture` ถูก copy-paste ไว้ในทุก layer

### วิธีแก้ — สร้าง Extension Methods ใน IIQE_UTILITY

```csharp
// IIQE_UTILITY/Extensions/ConversionExtensions.cs (สร้างใหม่)
namespace IIQE_UTILITY.Extensions
{
    public static class ConversionExtensions
    {
        /// <summary>
        /// แปลง string เป็น Int64 อย่างปลอดภัย คืนค่า 0 ถ้าแปลงไม่ได้
        /// </summary>
        public static long ToInt64Safe(this string value)
        {
            return long.TryParse(value, out long result) ? result : 0;
        }

        /// <summary>
        /// แปลง string เป็น int อย่างปลอดภัย คืนค่า 0 ถ้าแปลงไม่ได้
        /// </summary>
        public static int ToInt32Safe(this string value)
        {
            return int.TryParse(value, out int result) ? result : 0;
        }

        /// <summary>
        /// แปลง string เป็น bool อย่างปลอดภัย คืนค่า false ถ้าแปลงไม่ได้
        /// </summary>
        public static bool ToBoolSafe(this string value)
        {
            return bool.TryParse(value, out bool result) && result;
        }
    }
}
```

```csharp
// IIQE_UTILITY/Extensions/DataTableExtensions.cs (สร้างใหม่)
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;

namespace IIQE_UTILITY.Extensions
{
    public static class DataTableExtensions
    {
        /// <summary>
        /// แปลง DataTable เป็น List of T โดยใช้ Reflection
        /// (แทนที่ ConvertToList ที่ซ้ำกันทุก BL class)
        /// </summary>
        public static List<T> ToList<T>(this DataTable dt) where T : new()
        {
            var columnNames = dt.Columns.Cast<DataColumn>()
                .Select(c => c.ColumnName.ToLower())
                .ToList();
            var properties = typeof(T).GetProperties();

            return dt.AsEnumerable().Select(row =>
            {
                var obj = new T();
                foreach (var prop in properties)
                {
                    if (!columnNames.Contains(prop.Name.ToLower())) continue;
                    
                    var value = row[prop.Name];
                    if (value == DBNull.Value) continue;
                    
                    try
                    {
                        prop.SetValue(obj, value);
                    }
                    catch (Exception ex)
                    {
                        Logfile.Error($"DataTableExtensions.ToList: " +
                            $"Failed to set {prop.Name} = {value}: {ex.Message}");
                    }
                }
                return obj;
            }).ToList();
        }

        /// <summary>
        /// แปลง DataTable แถวแรกเป็น T
        /// </summary>
        public static T ToObject<T>(this DataTable dt) where T : new()
        {
            if (dt.Rows.Count == 0) return new T();
            return dt.ToList<T>().First();
        }
    }
}
```

```csharp
// IIQE_UTILITY/Extensions/CultureExtensions.cs (สร้างใหม่)
namespace IIQE_UTILITY.Extensions
{
    public static class CultureExtensions
    {
        /// <summary>
        /// เลือก text ตามภาษา (แทนที่ CheckCurrentCulture ที่ซ้ำทุก BL)
        /// </summary>
        public static string ByLanguage(this string lng, string thaiText, string englishText)
        {
            return lng == "en" ? englishText : thaiText;
        }
    }
}
```

### การ Migrate code เดิม

```diff
  // ก่อน — ConvertToInt64 ใน BL, Controller, Repo (ลบทิ้ง)
- private Int64 ConvertToInt64(string value) { ... }
- search.license_type_id = ConvertToInt64(arr[1]);
  
  // หลัง — ใช้ extension method
+ using IIQE_UTILITY.Extensions;
+ search.license_type_id = arr[1].ToInt64Safe();
```

```diff
  // ก่อน — ConvertToList ใน BL (ลบทิ้ง)
- corpExams = ConvertToList<CorpExamRequestList>(repo.GetData());

  // หลัง — ใช้ extension method
+ corpExams = repo.GetData().ToList<CorpExamRequestList>();
```

### Checklist ✅
- [ ] สร้าง `IIQE_UTILITY/Extensions/ConversionExtensions.cs`
- [ ] สร้าง `IIQE_UTILITY/Extensions/DataTableExtensions.cs`
- [ ] สร้าง `IIQE_UTILITY/Extensions/CultureExtensions.cs`
- [ ] ค่อยๆ migrate BL classes ให้ใช้ extension methods (ทีละไฟล์)
- [ ] ลบ private method `ConvertToInt64`, `ConvertToList`, `ConvertToObject` ที่ซ้ำ
- [ ] Build + test ทุกครั้งที่แก้ไฟล์

---

## Task 2.2: สร้าง Constants / Enums

### ปัญหา
Magic numbers เช่น status codes, role IDs เขียนตรงใน code ทั่วโปรเจกต์

### วิธีแก้ — สร้างไฟล์ Constants

```csharp
// IIQE_MODELS/Constants/FlowStatus.cs (สร้างใหม่)
namespace IIQE_MODELS.Constants
{
    /// <summary>
    /// สถานะ Flow ของคำขอสอบ — ค่าตรงกับตาราง mt_t_flowstatus
    /// </summary>
    public static class FlowStatus
    {
        // === สถานะทั่วไป ===
        public const int Draft = 0;
        public const int Created = 1;
        public const int PendingDraft = 7;
        public const int Submitted = 8;
        public const int Cancelled = 9;
        public const int PendingReview = 10;
        public const int ReturnedForEdit = 12;
        public const int PendingApproval = 13;
        public const int Delegated = 14;

        // === สถานะอนุมัติ ===
        public const int Approved = 27;
        public const int Rejected = 28;
        public const int ApprovedLevel2 = 35;
        public const int RejectedLevel2 = 36;

        // === สถานะยกเลิก ===
        public const int CancelledByMember = 20;
        public const int CancelledByAdmin = 24;
        public const int CancelledByCenter = 30;
        public const int CancelledByOIC = 34;
        public const int CancelledBySystem = 38;
        public const int CancelledExpired = 45;
        public const int CancelledDuplicate = 46;
        public const int CancelledOther = 48;
        public const int Completed = 51;
        public const int Closed = 55;

        // === กลุ่มสถานะ ===
        public static readonly int[] CancelledStatuses = 
            { 20, 24, 30, 34, 38, 45, 46, 48 };
        public static readonly int[] ActiveStatuses = 
            { 8, 10, 12, 13, 14, 15, 16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 29, 31, 32, 33, 35, 37, 41, 42, 43 };
        public static readonly int[] HistoryStatuses = 
            { 20, 24, 28, 30, 34, 36, 38, 45, 46, 48, 51, 55 };
    }
}
```

```csharp
// IIQE_MODELS/Constants/RoleType.cs (สร้างใหม่)
namespace IIQE_MODELS.Constants
{
    /// <summary>
    /// ประเภทผู้ใช้ — ค่าตรงกับ mt_t_role_type
    /// </summary>
    public static class RoleType
    {
        public const int PersonalMember = 1;
        public const int CorporateMember = 2;
        public const int ExamCenter = 3;
        public const int ProvinceOIC = 4;
        public const int Approver1 = 5;
        public const int Approver2 = 6;
        public const int Approver3 = 7;
        public const int Approver4 = 8;
        public const int FinalApprover = 9;
        public const int Reviewer1 = 10;
        public const int Reviewer2 = 11;
        public const int Reviewer3 = 12;
        public const int DelegateApprover = 13;
        public const int Outsource = 17;
        public const int SpecialApprover = 19;
    }
}
```

### การ Migrate code เดิม

```diff
  // ก่อน
- if (search.role_type_id == 1 || search.role_type_id == 2)
- else if (search.role_type_id == 3)
- SQL += " AND exam.status NOT IN (0, 20, 24, 28, 30, 34, 36, 38, 45, 46, 48) ";

  // หลัง
+ using IIQE_MODELS.Constants;
+ if (search.role_type_id == RoleType.PersonalMember || 
+     search.role_type_id == RoleType.CorporateMember)
+ else if (search.role_type_id == RoleType.ExamCenter)
+ SQL += $" AND exam.status NOT IN ({string.Join(",", FlowStatus.HistoryStatuses)}) ";
```

### Checklist ✅
- [ ] สร้าง `IIQE_MODELS/Constants/FlowStatus.cs`
- [ ] สร้าง `IIQE_MODELS/Constants/RoleType.cs`
- [ ] สร้าง `IIQE_MODELS/Constants/ErrorCode.cs` (สำหรับ popup_code)
- [ ] ค่อยๆ แทนที่ magic numbers ใน BL layer (ทีละไฟล์)
- [ ] ค่อยๆ แทนที่ magic numbers ใน DL layer
- [ ] ค่อยๆ แทนที่ magic numbers ใน Controllers

---

## Task 2.3: ใช้ IOptions&lt;T&gt; Pattern

### ปัญหา
Constructor ของ BL classes อ่าน `IConfiguration` ทีละค่า (60+ ค่าใน class เดียว)

### วิธีแก้ — สร้าง Configuration Classes

```csharp
// IIQE_MODELS/Configurations/ExternalApiSettings.cs (สร้างใหม่)
namespace IIQE_MODELS.Configurations
{
    public class ExternalApiSettings
    {
        public bool IsConnection { get; set; }
        public string API { get; set; }
    }

    public class BrokerValidateSettings
    {
        public bool IsConnection { get; set; }
        public string API { get; set; }
    }

    public class OicApiHeader
    {
        public string OicSysId { get; set; }
        public string OicAppId { get; set; }
        public string OicSecret { get; set; }
    }

    public class SendMailSettings
    {
        public string URL { get; set; }
        public string OicSysId { get; set; }
        public string OicAppId { get; set; }
        public string OicSecret { get; set; }
        public string From { get; set; }
        public string Subject { get; set; }
    }

    public class BillPaymentSettings
    {
        public bool IsEnabled { get; set; }
        public string UrlCreate { get; set; }
        public string UrlDownload { get; set; }
        public string UrlInsertDownload { get; set; }
        public string AppId { get; set; }
        public string ApiKey { get; set; }
    }

    public class FileUploadSettings
    {
        public string PathUpload { get; set; }
        public string PathUploadApi { get; set; }
        public string PathDownloadApi { get; set; }
        public string OicSysId { get; set; }
        public string OicAppId { get; set; }
        public string OicSecret { get; set; }
        public string AppUser { get; set; }
    }
}
```

### ลงทะเบียนใน Startup

```csharp
// OIC_IIQE/Startup.cs — เพิ่มใน ConfigureServices
services.Configure<ExternalApiSettings>(
    Configuration.GetSection("OIC-ExternalConnectAPI"));
services.Configure<SendMailSettings>(
    Configuration.GetSection("OIC_API_SENDMAIL"));
services.Configure<BillPaymentSettings>(
    Configuration.GetSection("OIC_API_BILLPAYMENT_CREATE"));
services.Configure<FileUploadSettings>(
    Configuration.GetSection("OIC_API_FILE_UPLOAD_DOWNLOAD"));
```

### ใช้งานใน BL Class

```diff
  // ก่อน — 60+ lines ใน constructor
- public IIQE_CorpExamRequestBL(IConfiguration configuration, ...)
- {
-     _PathEmail = configuration.GetSection("OIC_API_SENDMAIL")["URL"];
-     _Sendemail_oic_sys_id = configuration.GetSection("OIC_API_SENDMAIL")["oic-sys-id"];
-     // ... อีก 57 ค่า
- }

  // หลัง — สะอาดกว่ามาก
+ public IIQE_CorpExamRequestBL(
+     IOptions<SendMailSettings> mailSettings,
+     IOptions<BillPaymentSettings> billSettings,
+     IOptions<FileUploadSettings> fileSettings,
+     ...)
+ {
+     _mailSettings = mailSettings.Value;
+     _billSettings = billSettings.Value;
+     _fileSettings = fileSettings.Value;
+ }
```

> [!TIP]
> ไม่จำเป็นต้องทำทุก class พร้อมกัน — เริ่มจาก class ที่มี configuration มากที่สุดก่อน (IIQE_CorpExamRequestBL)

### Checklist ✅
- [ ] สร้าง Configuration classes ใน IIQE_MODELS
- [ ] ลงทะเบียน `services.Configure<T>()` ใน Startup
- [ ] Migrate IIQE_CorpExamRequestBL เป็น class แรก
- [ ] Migrate BL classes ที่เหลือทีละตัว
- [ ] Build + test ทุกครั้ง

---

## Task 2.4: แทนที่ Deprecated APIs

### ปัญหา
ใช้ `IHostingEnvironment` ที่ deprecated ตั้งแต่ .NET Core 3.0

### วิธีแก้ — Find and Replace

```diff
- using Microsoft.AspNetCore.Hosting;  // (ถ้าใช้แค่ IHostingEnvironment)
+ using Microsoft.AspNetCore.Hosting;  // IWebHostEnvironment อยู่ใน namespace เดียวกัน

- private readonly IHostingEnvironment _environment;
+ private readonly IWebHostEnvironment _environment;

- public SomeClass(IHostingEnvironment environment)
+ public SomeClass(IWebHostEnvironment environment)
```

### ไฟล์ที่ต้องแก้ (15 ไฟล์)

- `IIQE_BL/Managers/Frontend/Implementations/IIQE_CorpExamRequestBL.cs`
- `IIQE_BL/Managers/Frontend/Implementations/IIQE_ExamRequestBL.cs`
- `IIQE_BL/Managers/Frontend/Implementations/IIQE_RegisterPersonalMemberBL.cs`
- `IIQE_BL/Managers/Frontend/Implementations/IIQE_Report*BL.cs` (10 ไฟล์)
- `IIQE_BL/Managers/Backend/Implementations/IIQE_ImportPaymentBL.cs`
- `IIQE_BL/Managers/Backend/Implementations/IIQE_ReceiptBL.cs`
- `IIQE_BL/Managers/ExternalAPI/Implementations/IIQE_ExamRequestResultBL.cs`

### Checklist ✅
- [ ] Replace ทุกจุดที่ใช้ `IHostingEnvironment` → `IWebHostEnvironment`
- [ ] Build สำเร็จ ไม่มี warning

---

## Task 2.5: Standard API Response Envelope

### วิธีแก้ — สร้าง Response Wrapper

```csharp
// IIQE_MODELS/ApiResponse.cs (สร้างใหม่)
namespace IIQE_MODELS
{
    public class ApiResponse<T>
    {
        public bool Success { get; set; }
        public T Data { get; set; }
        public string Message { get; set; }
        public string ErrorCode { get; set; }
        public PaginationInfo Pagination { get; set; }

        public static ApiResponse<T> Ok(T data, string message = null) 
            => new ApiResponse<T> { Success = true, Data = data, Message = message };

        public static ApiResponse<T> Fail(string message, string errorCode = null)
            => new ApiResponse<T> { Success = false, Message = message, ErrorCode = errorCode };
    }

    public class PaginationInfo
    {
        public int PageNumber { get; set; }
        public int PageSize { get; set; }
        public int TotalCount { get; set; }
        public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    }
}
```

---

## Task 2.6: Global Exception Handling Middleware

```csharp
// IIQE_API/Middleware/GlobalExceptionMiddleware.cs (สร้างใหม่)
using IIQE_MODELS;
using IIQE_UTILITY;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System;
using System.Net;
using System.Threading.Tasks;

namespace IIQE_API.Middleware
{
    public class GlobalExceptionMiddleware
    {
        private readonly RequestDelegate _next;

        public GlobalExceptionMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (Exception ex)
            {
                Logfile.Error($"Unhandled Exception: {ex.Message}");
                Logfile.Error($"StackTrace: {ex.StackTrace}");

                context.Response.ContentType = "application/json";
                context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;

                var response = ApiResponse<object>.Fail(
                    "เกิดข้อผิดพลาดภายในระบบ กรุณาลองใหม่อีกครั้ง");

                await context.Response.WriteAsync(
                    JsonConvert.SerializeObject(response));
            }
        }
    }
}
```

ลงทะเบียนใน Startup:
```csharp
// IIQE_API/Startup.cs
app.UseMiddleware<GlobalExceptionMiddleware>(); // ← ใส่ก่อน middleware อื่น
```

### Checklist ✅
- [ ] สร้าง `ApiResponse<T>` class
- [ ] สร้าง `GlobalExceptionMiddleware`
- [ ] ลงทะเบียนใน Startup ก่อน middleware อื่น
- [ ] ทดสอบ — เมื่อ exception เกิดขึ้น ได้ JSON response format ที่ถูกต้อง
