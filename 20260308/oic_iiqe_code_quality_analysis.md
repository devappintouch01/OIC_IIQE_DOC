# 🔍 OIC IIQE System — Source Code Quality Analysis & Improvement Roadmap

**ระบบ:** OIC IIQE (Insurance Intermediary Qualification Examination)  
**เทคโนโลยี:** .NET Core MVC, Oracle Database  
**วันที่วิเคราะห์:** 8 มีนาคม 2569  

---

## 📊 สรุปภาพรวมโปรเจกต์

| โปรเจกต์ | ประเภท | จำนวนไฟล์ .cs (โดยประมาณ) | หมายเหตุ |
|---|---|---|---|
| **OIC_IIQE** | MVC Web Application (Frontend) | ~120 Controllers, ~280 Views | เว็บหลักสำหรับผู้ใช้งาน |
| **IIQE_API** | Web API | ~29 Controllers, 5 Middleware | REST API สำหรับระบบภายนอก |
| **IIQE_BL** | Business Logic Layer | ~83 BL classes | ชั้น Logic |
| **IIQE_DL** | Data Layer (Repository) | ~100+ Repository classes | ชั้น Data Access |
| **IIQE_MODELS** | Models & ViewModels | ~300 files | DTO / ViewModel |
| **IIQE_UTILITY** | Utility Classes | 8 files | Helper / Log |
| **Batch Processes** | Console Apps | 3 projects | Background jobs |

> [!IMPORTANT]
> ​ระบบนี้เป็นระบบ **มีขนาดใหญ่มาก** — ไฟล์ Business Logic บางไฟล์มีขนาดกว่า **440KB / 9,500+ บรรทัด** (เช่น `IIQE_CorpExamRequestBL.cs`) และ Data Layer repository บางไฟล์มีมากกว่า **11,800 บรรทัด** — สะท้อนถึงปัญหา **God Class** ที่ร้ายแรง

---

## 🚨 ปัญหาที่พบ (จัดเรียงตามความรุนแรง)

### ระดับ 1: วิกฤต (Critical) 🔴

#### 1.1 Silent Exception Swallowing — กลืน Exception โดยไม่ Log
**ความเสี่ยง:** ระบบเกิดข้อผิดพลาดแต่ไม่แสดงใน Log ทำให้ Debug ยากมาก

ตรวจพบ **มากกว่า 140 จุด** ที่เขียน `catch` แล้วไม่ทำอะไรเลย:

```csharp
// ❌ ตัวอย่างจาก IIQE_CorpExamRequestBL.cs
catch (Exception ex) { }

// ❌ ตัวอย่างจาก IIQE_ComfirmationExamRequestBL.cs (พบ 30+ จุดในไฟล์เดียว)
catch (Exception ex) { }

// ❌ ตัวอย่างจาก GetReceiptByReqID
catch (Exception ex)
{
    // ไม่ log, ไม่ return error, ไม่ throw — data หาย quietly
}
```

**ไฟล์ที่พบมากที่สุด:**
- `IIQE_ComfirmationExamRequestBL.cs` — **30+ จุด**
- `IIQE_CorpExamRequestBL.cs` — **หลายจุด**
- `IIQE_GroupExamRequestBL.cs` — **หลายจุด**
- `IIQE_ExamRequestBL.cs` — **หลายจุด**

---

#### 1.2 Hardcoded Log File Paths
**ความเสี่ยง:** Log จะ fail ทันทีบน server ที่ไม่มี drive C: หรือ path ที่กำหนด

```csharp
// ❌ จาก ClassLogfile.cs — hardcode path C:\LogFile ทุก method
Log.Logger = new LoggerConfiguration()
    .WriteTo.File(@"C:\LogFile\Information\Information_Log_.txt", ...)
    .CreateLogger();

Log.Logger = new LoggerConfiguration()
    .WriteTo.File(@"C:\LogFile\Error\Error_Log_.txt", ...)
    .CreateLogger();
```

**ปัญหาเพิ่มเติม:** ทุก method สร้าง `LoggerConfiguration` ใหม่ทุกครั้ง → memory leak, race condition

---

#### 1.3 Instance-level Shared State ใน Repository (Thread Safety)
**ความเสี่ยง:** Data corruption ในกรณี concurrent requests

```csharp
// ❌ จาก IIQE_CorpExamRequestRepo.cs
public class IIQE_CorpExamRequestRepo : IIQE_ICorpExamRequestRepo
{
    DataTable dt = new DataTable();   // ❌ shared instance field
    string SQL = "";                   // ❌ shared instance field
    ClassConnectDB conn = new ClassConnectDB();

    // ทุก method ใช้ dt และ SQL ตัวเดียวกัน
    // เมื่อ request มาพร้อมกัน → SQL/dt ถูก overwrite
}
```

**พบปัญหานี้ใน:** ทุก Repository class (~100+ ไฟล์)

---

#### 1.4 ไม่มี Unit Test เลย
**ความเสี่ยง:** ไม่มีการตรวจสอบ regression เมื่อแก้ไขโค้ด

- ไม่พบ Test project ในโซลูชัน
- ไม่มีไฟล์ `*Test*.cs`, `*Spec*.cs` ใดๆ
- Business Logic 83+ classes ไม่มีการทดสอบอัตโนมัติ

---

### ระดับ 2: สำคัญมาก (High) 🟠

#### 2.1 God Classes — ไฟล์ที่ใหญ่เกินไป
**ปัญหา:** ละเมิดหลัก Single Responsibility Principle อย่างรุนแรง

| ไฟล์ | ขนาด | บรรทัด | ความรุนแรง |
|---|---|---|---|
| `IIQE_CorpExamRequestRepo.cs` (DL) | 641 KB | ~11,858 | 🔴🔴🔴 |
| `IIQE_CorpExamRequestBL.cs` (BL) | 440 KB | ~9,516 | 🔴🔴🔴 |
| `IIQE_UserBL.cs` (BL) | 344 KB | — | 🔴🔴 |
| `IIQE_RegisterBL.cs` (BL) | 300 KB | — | 🔴🔴 |
| `IIQE_GroupExamRequestBL.cs` (BL) | 298 KB | — | 🔴🔴 |
| `IIQE_RegisterPersonalMemberBL.cs` (BL) | 223 KB | — | 🔴🔴 |
| `IIQE_PersonalExamRoundBL.cs` (BL) | 200 KB | — | 🔴🔴 |
| `IIQE_ReceiptBL.cs` (BL) | 181 KB | — | 🔴 |
| `CorpExamRequestController.cs` (Web) | 102 KB | ~2,410 | 🔴 |
| `GroupExamRequestController.cs` (Web) | 97 KB | — | 🔴 |

---

#### 2.2 Code Duplication แพร่กระจาย
**ตัวอย่าง:** `ConvertToInt64` ถูกทำซ้ำใน **ทุกชั้น**

```csharp
// ปรากฏซ้ำใน: CorpExamRequestController, IIQE_CorpExamRequestBL, 
//             IIQE_CorpExamRequestRepo, และอีกนับสิบไฟล์
private Int64 ConvertToInt64(string value)
{
    Int64 _value;
    try { _value = Convert.ToInt64(value); }
    catch { _value = 0; }
    return _value;
}
```

**ตัวอย่างอื่น:**
- `ConvertToList<T>()` และ `ConvertToObject<T>()` ถูกคัดลอกในหลาย BL classes
- `CheckCurrentCulture()` ซ้ำในหลาย BL classes
- Pagination logic ซ้ำกันใน Controller ทุกตัว
- Role-based routing logic (`if IsExternal ... else`) ซ้ำทุก method ใน Controller

---

#### 2.3 Deprecated API Usage
**ปัญหา:** ใช้ API ที่ถูก deprecated แล้ว

```csharp
// ❌ IHostingEnvironment (deprecated ตั้งแต่ .NET Core 3.0)
// พบใน 15+ ไฟล์
private readonly IHostingEnvironment _environment;
// ✅ ควรใช้ IWebHostEnvironment

// ❌ RestSharp deprecated methods
var client = new RestClient(fullUrl);
client.Timeout = -1;  // deprecated
var request = new RestRequest(Method.GET);  // deprecated enum syntax
```

---

#### 2.4 Synchronous I/O ทั้งหมด — ไม่มี Async/Await
**ปัญหา:** I/O operations ทั้ง Database calls, HTTP calls, File operations เป็น synchronous ทั้งหมด

```csharp
// ❌ Database calls - synchronous 
dt.Load(Com.ExecuteReader());       // ควรเป็น ExecuteReaderAsync
Com.ExecuteNonQuery();              // ควรเป็น ExecuteNonQueryAsync

// ❌ HTTP calls - synchronous
IRestResponse response = client.Execute(request);  // ควรเป็น ExecuteAsync

// ❌ File operations - synchronous
File.Copy(sourceFile, destFile, true);  // ควรเป็น CopyAsync
```

**ผลกระทบ:** Thread pool starvation ภายใต้ high load → ASP.NET Core timeout

---

### ระดับ 3: ปานกลาง (Medium) 🟡

#### 3.1 API Middleware Configuration ที่ซ้ำซ้อนและขัดแย้ง

```csharp
// ❌ จาก IIQE_API Startup.cs — middleware ถูก apply ซ้ำซ้อน
// AuthReceiptMiddleware ถูก apply 2 ครั้ง (line 89th & 101st)
app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/Receipt"), ...);
app.UseWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api/Receipt"), ...);

// ซ้ำอีกรอบ!
app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/Receipt/C_Receipt"), ...);
app.UseWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api/Receipt/C_Receipt"), ...);

// Middleware ถูก register หลัง UseEndpoints (line 123) — จะไม่ทำงาน!
app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/AgentRound"), ...);
```

---

#### 3.2 Constructor Over-injection (Massive Constructors)

```csharp
// ❌ Constructor รับ 9+ dependencies — สัญญาณของ God Class
public IIQE_CorpExamRequestBL(
    IIQE_ICorpExamRequestRepo corpExamRequestRepo,
    IConfiguration configuration,
    IIQE_IBillPaymentBL billBL,
    IIQE_ISendEmailBL sentmail,
    IIQE_IReceiptBL _IReceiptBL,
    IIQE_IExamRequestBL examRequest,
    IIQE_IBillPaymentRepo billRepo,
    IHostingEnvironment environment,
    IIQE_ILogsRepo logsRepo)
{
    // อ่าน configuration 60+ ค่า ในconstructor...
}
```

---

#### 3.3 DI Registration ซ้ำซ้อน

```csharp
// ❌ จาก Binder.cs — registered ซ้ำ
services.AddScoped<IIQE_IExamCenterBL, IIQE_ExamCenterBL>();  // line 40
services.AddScoped<IIQE_IExamCenterBL, IIQE_ExamCenterBL>();  // line 46 ซ้ำ!

services.AddScoped<IIQE_ApproveExamResultLicenseBL>();  // line 121
services.AddScoped<IIQE_ApproveExamResultLicenseBL>();  // line 165 ซ้ำ!
```

---

#### 3.4 Magic Numbers / Strings แทรกอยู่ทั่ว

```csharp
// ❌ Magic numbers สำหรับ status
if (exam.status == 14 && exam.delegate_status == 2)
AND id NOT IN (9, 20, 24, 30, 34, 38, 45, 46, 48, 28, 36)
exam.status IN (20, 24, 28, 30, 34, 38, 45, 46, 48, 51, 55)

// ❌ Magic numbers สำหรับ role
if (search.role_type_id == 1 || search.role_type_id == 2)
else if (search.role_type_id == 3)

// ❌ Magic string for error codes
param.message_code = "0189";
param.message_code = "0415";
param.message_code = "0249";
```

---

#### 3.5 Configuration อ่านตรงใน Constructor — 60+ ค่า

```csharp
// ❌ อ่าน configuration ค่าทีละตัว ใน constructor ยาวมาก
_PathEmail = configuration.GetSection("OIC_API_SENDMAIL")["URL"];
_Sendemail_oic_sys_id = configuration.GetSection("OIC_API_SENDMAIL")["oic-sys-id"];
// ... อีก 57 ค่า
```

**ควรใช้:** `IOptions<T>` pattern กับ strongly-typed configuration classes

---

#### 3.6 Naming Convention ไม่สม่ำเสมอ

```
// ❌ ชื่อสะกดผิด
BlcklistSettingController.cs  VS  BlocklistSettingController.cs  (ทั้ง 2 มีอยู่!)
IIQE_ComfirmationExamRequestBL.cs (ควรเป็น Confirmation)
IsVadidate (ควรเป็น IsValidate)

// ❌ Naming convention ไม่สม่ำเสมอ
_Url_URL_URL (ชื่อตัวแปรที่ทำให้สับสนมาก)
_Url_Broker_URL vs UrlBrokerValidate
_file_oic_secet (ควรเป็น secret)
```

---

### ระดับ 4: ควรปรับปรุง (Low) 🟢

#### 4.1 ไม่มี Global Error Handling

```csharp
// Web App — ใช้แค่ UseExceptionHandler
app.UseExceptionHandler("/Home/Error");

// API — ไม่มี global exception middleware
// แต่ละ controller catch เอง (และหลายจุดไม่ catch)
```

---

#### 4.2 Commented-out Code ทิ้งไว้ทั่วโปรเจกต์

```csharp
// ❌ พบ commented code จำนวนมาก เช่น:
//if (IsExternal)
//if (IsExternal)
result = _service.GetErrorMessage(param);

/*old
if (search.search_type == 1)
    SQL += " AND exam.status NOT IN (0, 20, 24, 28, 30, 34, 36, 38, 45, 46, 48) ";
*/

//services.AddOcelot();
//app.UseOcelot();
```

---

#### 4.3 UseHttpsRedirection ถูกเรียก 2 ครั้ง

```csharp
// ❌ OIC_IIQE Startup.cs
app.UseHttpsRedirection();  // line 131
// ... code ...
app.UseHttpsRedirection();  // line 137 — ซ้ำ!
```

---

#### 4.4 DataTable เป็น Return Type ระหว่าง Layer

```csharp
// ❌ Repository return DataTable → BL convert เป็น List<T>
// ทำให้ BL ต้องรู้โครงสร้าง column ของ database
public DataTable GetMasterLicenseType(string lng) { ... }
// แล้ว BL เรียก ConvertToList<T>() ซึ่งใช้ Reflection
```

---

#### 4.5 ไม่มี Response Envelope มาตรฐานสำหรับ API

```csharp
// ❌ API Controllers return แตกต่างกันไป
return Json(result);
return Json(new { data = result });
return Json(new { isSuccess = model.is_success, message = model.message });
return Json(new { dataList = models });
```

---

## 🗺️ Roadmap การแก้ไข

### Phase 1: Critical Fixes — ต้องทำทันที (สัปดาห์ที่ 1-4)

| # | Task | ไฟล์ที่เกี่ยวข้อง | ความเสี่ยงถ้าไม่ทำ | ระดับ effort |
|---|---|---|---|---|
| 1.1 | **แก้ Silent Exception Swallowing** — เพิ่ม logging ทุกจุดที่ `catch (Exception ex) { }` ขั้นต่ำใส่ `Logfile.Error(ex.Message)` | BL Layer ทั้งหมด (~140 จุด) | ระบบ error แต่ไม่มี log → debug ไม่ได้ | 🟡 Medium |
| 1.2 | **แก้ Hardcoded Log Paths** — ย้าย path log ไป `appsettings.json`, สร้าง Logger ครั้งเดียวแบบ Singleton | `ClassLogfile.cs`, `LogHelper.cs` | Log fail บน production server | 🟢 Low |
| 1.3 | **แก้ Thread Safety ใน Repository** — เปลี่ยน field-level `DataTable dt` และ `string SQL` เป็น local variable ใน method | DL Layer ทุก Repository (~100+ ไฟล์) | Data corruption / random bugs | 🟠 High |
| 1.4 | **แก้ API Middleware Order** — จัดเรียง middleware ใหม่, ลบ duplicates, ย้าย middleware ที่อยู่หลัง `UseEndpoints` ขึ้นมาก่อน | `IIQE_API/Startup.cs` | Middleware ไม่ทำงาน, security bypass | 🟢 Low |

---

### Phase 2: Architecture Improvements (สัปดาห์ที่ 5-12)

| # | Task | รายละเอียด | ระดับ effort |
|---|---|---|---|
| 2.1 | **สร้าง Shared Utility Library** — รวม `ConvertToInt64`, `ConvertToList<T>`, `CheckCurrentCulture` เป็น static extension methods | ลด duplication, maintain ง่ายขึ้น | 🟡 Medium |
| 2.2 | **แทนที่ IHostingEnvironment** ด้วย `IWebHostEnvironment` | อัปเดต deprecated API (15+ ไฟล์) | 🟢 Low |
| 2.3 | **สร้าง Constants Class** — แทนที่ magic numbers ด้วย named constants | สร้าง `FlowStatus`, `RoleType`, `ErrorCode` enum/constants | 🟡 Medium |
| 2.4 | **ใช้ Options Pattern** — สร้าง strongly-typed configuration classes | แทนที่ `configuration["..."]` 60+ ตัวใน constructor | 🟡 Medium |
| 2.5 | **ลบ Commented Code** ทั้งหมด — ใช้ Git history แทน | ทำให้โค้ดอ่านง่ายขึ้น | 🟢 Low |
| 2.6 | **แก้ DI Registration ซ้ำซ้อน** | ลบ duplicate registrations ใน `Binder.cs` | 🟢 Low |
| 2.7 | **แก้ Naming Convention** — rename ไฟล์ที่สะกดผิด | `Comfirmation` → `Confirmation`, `Blcklist` → `Blocklist` | 🟢 Low |

---

### Phase 3: Code Refactoring (สัปดาห์ที่ 13-24)

| # | Task | รายละเอียด | ระดับ effort |
|---|---|---|---|
| 3.1 | **แตก God Classes** — เริ่มจากไฟล์ที่ใหญ่ที่สุด | แยก `IIQE_CorpExamRequestBL` (9,500 บรรทัด) เป็น domain-specific BLs เช่น `CorpExamRound`, `CorpExamAttendee`, `CorpExamApproval` | 🔴 Very High |
| 3.2 | **เปลี่ยน Return Type จาก DataTable เป็น Model** | Repository ควร return `List<T>` / `T` แทน `DataTable` | 🔴 Very High |
| 3.3 | **เพิ่ม Async/Await** — เริ่มจาก API Layer และ critical paths | DB calls, HTTP calls, File I/O | 🟠 High |
| 3.4 | **ลด Controller Code Duplication** — สร้าง Base Controller class | รวม pagination logic, role checking, culture detection | 🟡 Medium |
| 3.5 | **สร้าง Global Exception Handling Middleware** | จัดการ error response แบบ centralized | 🟡 Medium |
| 3.6 | **สร้าง Standard API Response Envelope** | `{ success, data, errors, pagination }` | 🟡 Medium |

---

### Phase 4: Quality & Testing (สัปดาห์ที่ 25-36)

| # | Task | รายละเอียด | ระดับ effort |
|---|---|---|---|
| 4.1 | **สร้าง Unit Test Project** | เพิ่ม xUnit/NUnit project + test BL layer | 🟠 High |
| 4.2 | **สร้าง Integration Tests** สำหรับ critical API endpoints | Test ที่ cover business flows หลัก | 🟠 High |
| 4.3 | **เพิ่ม Code Analyzer** (StyleCop, SonarAnalyzer) | Enforce coding standards อัตโนมัติ | 🟡 Medium |
| 4.4 | **Migrate to Dapper / EF Core** (Long-term) | แทนที่ raw SQL string building ด้วย micro-ORM | 🔴 Very High |
| 4.5 | **เพิ่ม Health Check Endpoints** | Monitor สุขภาพระบบ | 🟢 Low |
| 4.6 | **เพิ่ม API Documentation** (Swagger) สำหรับ Production | ลบ `#if DEBUG` wrapper ออกจาก Swagger | 🟢 Low |

---

## 📈 Priority Matrix

```
                    ผลกระทบสูง
                        │
     [1.3 Thread Safety]│[1.1 Exception Swallowing]
     [3.1 God Classes]  │[1.2 Hardcoded Logs]
                        │[1.4 Middleware Order]
   ─────────────────────┼─────────────────────
     [3.2 DataTable]    │[2.1 Shared Utils]
     [4.4 ORM]          │[2.3 Constants]
     [3.3 Async]        │[2.4 Options Pattern]
                        │[4.1 Unit Tests]
                        │
                    ผลกระทบต่ำ
    Effort สูง ←────────┼────────→ Effort ต่ำ
```

---

## 🎯 Quick Wins — สิ่งที่ทำได้ทันทีโดยไม่กระทบระบบ

1. ✅ **เพิ่ม logging ใน catch blocks ว่างๆ** — ไม่เปลี่ยน flow, แค่เพิ่ม visibility
2. ✅ **ลบ commented-out code** — ใช้ Git history แทน
3. ✅ **แก้ UseHttpsRedirection ซ้ำ** — ลบบรรทัดซ้ำ 1 บรรทัด
4. ✅ **แก้ DI registration ซ้ำ** — ลบบรรทัดซ้ำ
5. ✅ **ย้าย log path ไป configuration** — เปลี่ยนแค่ `ClassLogfile.cs`

---

## 📋 สรุปสถิติปัญหา

| หมวดหมู่ | จำนวนที่พบ | ระดับ |
|---|---|---|
| Silent Exception Catch | 140+ จุด | 🔴 Critical |
| God Class (>5,000 lines) | 6+ ไฟล์ | 🟠 High |
| Deprecated API Usage | 15+ ไฟล์ | 🟡 Medium |
| Code Duplication | ทั่วโปรเจกต์ | 🟠 High |
| Thread Safety Issue | 100+ Repository files | 🔴 Critical |
| Magic Numbers/Strings | ทั่วโปรเจกต์ | 🟡 Medium |
| Naming Inconsistency | 5+ ไฟล์ | 🟢 Low |
| No Unit Tests | ทั้งระบบ | 🔴 Critical |
| Hardcoded Paths | Utility layer | 🔴 Critical |
| Middleware Misconfiguration | API Startup | 🟡 Medium |

---

> [!TIP]
> **คำแนะนำ:** ควรเริ่มจาก **Phase 1 (Critical Fixes)** ก่อน เพราะส่วนใหญ่เป็น low-effort แต่ high-impact สามารถทำเสร็จภายใน 1-4 สัปดาห์โดยไม่กระทบ business logic ที่มีอยู่

> [!WARNING]  
> **Phase 3 — God Class Refactoring** ควรทำอย่างระมัดระวังเป็นพิเศษ เพราะไฟล์อย่าง `IIQE_CorpExamRequestBL.cs` (9,500+ บรรทัด) มี business logic ที่ซับซ้อนมาก การ refactor ต้องมี integration test ครอบคลุมก่อนเริ่มแยก class เพื่อป้องกัน regression
