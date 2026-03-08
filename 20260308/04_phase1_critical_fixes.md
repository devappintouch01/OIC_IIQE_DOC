# 🔧 คู่มือแก้ไข Phase 1: Critical Fixes

**ระยะเวลา:** สัปดาห์ที่ 1-4  
**เป้าหมาย:** แก้ปัญหาที่กระทบ stability, debuggability, thread safety  
**ความเสี่ยง:** ต่ำ — ส่วนใหญ่ไม่เปลี่ยน business logic

---

## Task 1.1: แก้ Silent Exception Swallowing (140+ จุด)

### ปัญหา
Exception ถูก catch แล้วไม่ทำอะไร ทำให้ debug ไม่ได้เมื่อเกิดปัญหา

### ไฟล์ที่ต้องแก้ (เรียงตามปริมาณปัญหา)

| ไฟล์ | จำนวนจุดที่ต้องแก้ |
|---|---|
| `IIQE_BL/Managers/Frontend/Implementations/IIQE_ComfirmationExamRequestBL.cs` | ~30 จุด |
| `IIQE_BL/Managers/Frontend/Implementations/IIQE_CorpExamRequestBL.cs` | ~10 จุด |
| `IIQE_BL/Managers/Frontend/Implementations/IIQE_GroupExamRequestBL.cs` | ~5 จุด |
| `IIQE_BL/Managers/Frontend/Implementations/IIQE_ExamRequestBL.cs` | ~5 จุด |
| `IIQE_BL/Managers/Frontend/Implementations/IIQE_ExamScheduleBL.cs` | ~3 จุด |
| `IIQE_BL/Managers/Login/Implementations/IIQE_LoginBL.cs` | ~3 จุด |
| `IIQE_BL/Managers/Backend/Implementations/` (หลายไฟล์) | ~15 จุด |
| `IIQE_BL/Managers/Frontend/Implementations/` (ที่เหลือ) | ~70 จุด |

### วิธีแก้ — ระดับ 1: เพิ่ม Logging (ทำก่อน)

```diff
// ก่อนแก้
  catch (Exception ex) { }

// หลังแก้ — เพิ่ม logging ขั้นต่ำ
+ catch (Exception ex) 
+ { 
+     Logfile.Error($"{nameof(ClassName)}.{nameof(MethodName)}: {ex.Message}");
+ }
```

**ตัวอย่างจริง — IIQE_CorpExamRequestBL.cs:**

```diff
  public List<ReceiptGroup> GetReceiptByReqID(int REQ_ID)
  {
      List<ReceiptGroup> Examiner_List = new List<ReceiptGroup>();
      try
      {
          Examiner_List = ConvertToList<ReceiptGroup>(
              _ICorpExamRequestRepo.GetReceiptByReqID(REQ_ID));
      }
-     catch (Exception ex)
-     {
-     }
+     catch (Exception ex)
+     {
+         Logfile.Error($"IIQE_CorpExamRequestBL.GetReceiptByReqID(REQ_ID={REQ_ID}): {ex.Message}");
+     }
      return Examiner_List;
  }
```

### วิธีแก้ — ระดับ 2: เพิ่ม Structured Logging (ทำตาม)

```diff
  // สำหรับ method ที่ return data — log + return empty
  catch (Exception ex)
  {
+     Logfile.Error($"{nameof(IIQE_CorpExamRequestBL)}.{nameof(GetReceiptByReqID)}: " +
+         $"REQ_ID={REQ_ID}, Error={ex.Message}, StackTrace={ex.StackTrace}");
  }
  
  // สำหรับ method ที่ทำ mutation (Insert/Update/Delete) — ควร throw
  catch (Exception ex)
  {
+     Logfile.Error($"{nameof(InsertExamAtchFromCorpMember)}: " +
+         $"exam_req_id={exam_req_id}, Error={ex.Message}");
+     throw; // ← ให้ caller จัดการ
  }
```

### วิธีค้นหาทั้งหมด

ค้นหาใน Visual Studio / VS Code:
```
Regex search: catch\s*\(Exception\s+\w+\)\s*\{\s*\}
Path: IIQE_BL/
```

หรือใช้ PowerShell:
```powershell
Get-ChildItem -Path "d:\Works\OICIIQE\IIQE_BL" -Filter "*.cs" -Recurse | 
    Select-String -Pattern "catch\s*\(Exception\s+\w+\)\s*\{\s*\}" | 
    Select-Object Filename, LineNumber
```

### Checklist ✅
- [ ] แก้ทุกจุดใน `IIQE_ComfirmationExamRequestBL.cs` (~30 จุด)
- [ ] แก้ทุกจุดใน `IIQE_CorpExamRequestBL.cs`
- [ ] แก้ทุกจุดใน BL Frontend Implementations ที่เหลือ
- [ ] แก้ทุกจุดใน BL Backend Implementations
- [ ] ตรวจสอบว่า log file ถูกเขียนหลังแก้
- [ ] แก้ `catch { }` (ไม่มี Exception variable) ด้วย — ค้นหา `catch\s*\{\s*\}`

---

## Task 1.2: แก้ Hardcoded Log Paths + Logger Singleton

### ปัญหา
- `ClassLogfile.cs` hardcode path `C:\LogFile\` 
- สร้าง `LoggerConfiguration` ใหม่ทุกครั้งที่เรียก → memory leak + race condition

### ไฟล์ที่ต้องแก้
- `IIQE_UTILITY/ClassLogfile.cs`
- `IIQE_UTILITY/LogHelper.cs`
- ทุก `appsettings.json` (เพิ่ม log config section)

### วิธีแก้ — สร้าง ClassLogfile ใหม่

```csharp
// IIQE_UTILITY/ClassLogfile.cs — หลังแก้
using Microsoft.Extensions.Configuration;
using Serilog;
using Serilog.Core;
using System;
using System.IO;

namespace IIQE_UTILITY
{
    public class Logfile
    {
        private static readonly object _lock = new object();
        private static bool _isInitialized = false;
        private static string _basePath = @"C:\LogFile"; // fallback default

        /// <summary>
        /// Initialize logger with configuration (ต้องเรียก 1 ครั้งตอน startup)
        /// </summary>
        public static void Initialize(IConfiguration configuration)
        {
            if (_isInitialized) return;
            lock (_lock)
            {
                if (_isInitialized) return;

                _basePath = configuration["Logging:FilePath"] ?? @"C:\LogFile";
                
                // สร้าง directory ถ้ายังไม่มี
                Directory.CreateDirectory(_basePath);

                Log.Logger = new LoggerConfiguration()
                    .MinimumLevel.Debug()
                    .WriteTo.File(
                        path: Path.Combine(_basePath, "Application", "Log_.txt"),
                        fileSizeLimitBytes: 10_485_760, // 10MB (แทน 100KB)
                        rollOnFileSizeLimit: true,
                        rollingInterval: RollingInterval.Day,
                        retainedFileCountLimit: 30,
                        shared: true,
                        outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff} [{Level:u3}] {Message:lj}{NewLine}{Exception}")
                    .CreateLogger();

                _isInitialized = true;
            }
        }

        public static void Information(string msg) => Log.Information(msg);
        public static void Error(string msg) => Log.Error(msg);
        public static void Error(string msg, Exception ex) => Log.Error(ex, msg);
        public static void Warning(string msg) => Log.Warning(msg);
    }
}
```

### เพิ่ม Configuration ใน appsettings.json

```json
{
  "Logging": {
    "FilePath": "D:\\Logs\\IIQE"
  }
}
```

### เรียก Initialize ตอน Startup

```csharp
// OIC_IIQE/Program.cs หรือ Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // เพิ่มบรรทัดนี้ตอนเริ่มต้น
    Logfile.Initialize(Configuration);
    // ... code เดิม
}
```

### Checklist ✅
- [ ] แก้ไข `ClassLogfile.cs` ตาม template ใหม่
- [ ] เพิ่ม `Logging:FilePath` ใน `appsettings.json` ทุกโปรเจกต์
- [ ] เรียก `Logfile.Initialize()` ใน `OIC_IIQE/Startup.cs`
- [ ] เรียก `Logfile.Initialize()` ใน `IIQE_API/Startup.cs`
- [ ] เรียก `Logfile.Initialize()` ใน Batch Process ทุกตัว
- [ ] ทดสอบว่า log file ถูกเขียนตาม path ใหม่
- [ ] ตรวจสอบว่า log file size limit เป็น 10MB

---

## Task 1.3: แก้ Thread Safety ใน Repository

### ปัญหา
ทุก Repository ใช้ `DataTable dt` และ `string SQL` เป็น instance field — เมื่อ concurrent requests มา field เหล่านี้จะถูก overwrite

### ไฟล์ที่ต้องแก้
**ทุก Repository class ใน `IIQE_DL/Repositories/`** (~100+ ไฟล์)

### วิธีแก้ — เปลี่ยน instance field เป็น local variable

```diff
  public class IIQE_CorpExamRequestRepo : IIQE_ICorpExamRequestRepo
  {
-     DataTable dt = new DataTable();
-     string SQL = "";
-     ClassConnectDB conn = new ClassConnectDB();
+     private readonly ClassConnectDB conn = new ClassConnectDB();
      private string _configuration;

      // ทุก method ที่ใช้ dt และ SQL ต้องประกาศ local variable
      public DataTable GetMasterLicenseType(string lng)
      {
+         string SQL;
          if (lng == "en")
          {
              SQL = @"SELECT ...";
          }
          else
          {
              SQL = @"SELECT ...";
          }
-         dt = conn.ExecuteReaderWithParams(SQL, _configuration, null);
-         return dt;
+         return conn.ExecuteReaderWithParams(SQL, _configuration, null);
      }
  }
```

### ขั้นตอนสำหรับแต่ละ Repository file

1. **ลบ** instance fields: `DataTable dt = new DataTable();` และ `string SQL = "";`
2. **เปลี่ยน** `ClassConnectDB conn` เป็น `private readonly ClassConnectDB conn`
3. สำหรับ**ทุก method**:
   - เพิ่ม `string SQL;` หรือ `string SQL = "";` เป็น local variable
   - เปลี่ยน `dt = conn.Execute...(...)` → `DataTable dt = conn.Execute...(...)` หรือ return ตรง
   - ตรวจสอบว่า method ไม่ได้ใช้ `dt` ข้าม method

### วิธีค้นหาไฟล์ที่ต้องแก้

```powershell
Get-ChildItem -Path "d:\Works\OICIIQE\IIQE_DL\Repositories" -Filter "*.cs" -Recurse | 
    Select-String -Pattern "^\s+(DataTable dt|string SQL\s*=)" | 
    Select-Object Filename -Unique
```

### ตัวอย่างจริง — GetErrorMessage

```diff
  public String GetErrorMessage(string lng, string popup_code)
  {
      string str = "";
+     string SQL;
      OracleParameter[] parameterArray = new OracleParameter[1];
      if (lng == "en")
          SQL = "SELECT TEXT_EN AS TEXT_ FROM MT_T_ERROR_MESSAGE WHERE POPUP_CODE = :POPUP_CODE ";
      else
          SQL = "SELECT TEXT_TH AS TEXT_ FROM MT_T_ERROR_MESSAGE WHERE POPUP_CODE = :POPUP_CODE ";

      OracleParameter p0 = new OracleParameter("POPUP_CODE", OracleDbType.Varchar2);
      p0.Direction = ParameterDirection.Input;
      p0.Value = popup_code;
      parameterArray[0] = p0;
-     dt = conn.ExecuteReaderWithParams(SQL, _configuration, parameterArray);
+     DataTable dt = conn.ExecuteReaderWithParams(SQL, _configuration, parameterArray);
      if (dt.Rows.Count > 0)
      {
          str = dt.Rows[0][0].ToString();
      }
      return str;
  }
```

> [!WARNING]
> **สำคัญ:** ต้องแก้ทีละไฟล์ แล้ว build ทดสอบทุกครั้ง เพราะบาง method อาจใช้ `dt` จาก method อื่น (แม้ว่าจะเป็น anti-pattern แต่อาจมี dependency ที่ซ่อนอยู่)

### Checklist ✅
- [ ] แก้ Repository ใน `IIQE_DL/Repositories/Frontend/Implementations/` (51 files)
- [ ] แก้ Repository ใน `IIQE_DL/Repositories/Backend/` (หลายไฟล์)
- [ ] แก้ Repository ใน `IIQE_DL/Repositories/Login/`
- [ ] แก้ Repository ใน `IIQE_DL/Repositories/Shared/`
- [ ] แก้ Repository อื่นๆ
- [ ] Build สำเร็จ ไม่มี error
- [ ] ทดสอบ concurrent requests — ข้อมูลไม่สลับกัน

---

## Task 1.4: แก้ API Middleware Pipeline

### ปัญหา
1. Middleware ซ้ำซ้อน (AuthReceipt ลงทะเบียน 2 ครั้ง)
2. Middleware ลงทะเบียนหลัง `UseEndpoints` (จะไม่ทำงาน)

### ไฟล์ที่ต้องแก้
- `IIQE_API/Startup.cs`

### วิธีแก้

```diff
  public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
  {
      if (env.IsDevelopment())
      {
          app.UseDeveloperExceptionPage();
      }

      app.UseHttpsRedirection();
      app.UseCors("AllowSpecificOrigins");
      app.UseRouting();
      app.UseAuthorization();

+     // Authentication middleware — จัดกลุ่มให้ชัดเจน
      app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/E_LicensingExamAPI"), _appBuilder =>
      {
          _appBuilder.UseMiddleware<AuthE_LicensingMiddleware>();
      });

      app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/ExamResult"), _appBuilder =>
      {
          _appBuilder.UseMiddleware<AuthMiddleware>();
      });

+     app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/AgentRound"), _appBuilder =>
+     {
+         _appBuilder.UseMiddleware<AuthMiddleware>();
+     });

-     app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/Receipt"), _appBuilder =>
-     {
-         _appBuilder.UseMiddleware<AuthReceiptMiddleware>();
-     });
-     app.UseWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api/Receipt"), _appBuilder =>
-     {
-         _appBuilder.UseMiddleware<ApiKeyMiddleware>();
-     });
-     app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/Receipt/C_Receipt"), _appBuilder =>
-     {
-         _appBuilder.UseMiddleware<AuthReceiptMiddleware>();
-     });
-     app.UseWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api/Receipt/C_Receipt"), _appBuilder =>
-     {
-         _appBuilder.UseMiddleware<ApiKeyMiddleware>();
-     });

+     // Receipt endpoints ใช้ AuthReceiptMiddleware
+     app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/Receipt"), _appBuilder =>
+     {
+         _appBuilder.UseMiddleware<AuthReceiptMiddleware>();
+     });
+
+     // ที่เหลือทั้งหมดใช้ ApiKeyMiddleware
+     app.UseWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api/Receipt"), _appBuilder =>
+     {
+         _appBuilder.UseMiddleware<ApiKeyMiddleware>();
+     });

  #if DEBUG
      app.UseSwagger();
      app.UseSwaggerUI(...);
  #endif

      app.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
      });

-     // ❌ ลบออก — middleware หลัง UseEndpoints จะไม่ทำงาน
-     app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api/AgentRound"), _appBuilder =>
-     {
-         _appBuilder.UseMiddleware<AuthMiddleware>();
-     });
  }
```

### Checklist ✅
- [ ] ลบ middleware registration ที่ซ้ำ
- [ ] ย้าย AgentRound middleware ขึ้นมาก่อน UseEndpoints
- [ ] ทดสอบ `/api/Receipt` → ยังทำงานถูกต้อง
- [ ] ทดสอบ `/api/AgentRound` → middleware ทำงาน
- [ ] ทดสอบ endpoints อื่นๆ → ApiKey middleware ทำงาน

---

## Task 1.5: Cleanup — ลบ Commented Code, ลบ DI ซ้ำ

### 1.5a ลบ UseHttpsRedirection ซ้ำ

**ไฟล์:** `OIC_IIQE/Startup.cs`

```diff
  app.UseHttpsRedirection();
  app.UseStaticFiles();
  app.UseSession();
  app.UseRouting();
- app.UseHttpsRedirection();  // ← ลบบรรทัดนี้
  app.UseCors("AllowSpecificOrigins");
```

### 1.5b ลบ DI Registration ที่ซ้ำ

**ไฟล์:** `IIQE_BL/Binder.cs`

```diff
  services.AddScoped<IIQE_IExamCenterBL, IIQE_ExamCenterBL>();
  // ...
- services.AddScoped<IIQE_IExamCenterBL, IIQE_ExamCenterBL>();   // ลบซ้ำ (line ~46)

  // ...
- services.AddScoped<IIQE_ApproveExamResultLicenseBL>();          // ลบซ้ำ (line ~165)

  // ...
- services.AddScoped<IIQE_IYearlyHolidayFrontendBL, ...>();       // ลบซ้ำ (line ~176)
```

**ไฟล์:** `IIQE_DL/Binder.cs`

```diff
  services.AddScoped<IIQE_IExamCenterRepo, IIQE_ExamCenterRepo>();
  // ...
- services.AddScoped<IIQE_IExamCenterRepo, IIQE_ExamCenterRepo>();  // ลบซ้ำ (line ~43)
```

### 1.5c ลบ Commented-out Code (เลือกทำ)

ลบ commented-out code ที่ยาวๆ เช่น:
- `OIC_IIQE/Startup.cs` — ~25 บรรทัด commented localization code
- `IIQE_API/Startup.cs` — commented Ocelot code

> [!TIP]
> ใช้ Git history เมื่อต้องการดู code ที่ลบไป ไม่ต้อง comment ทิ้งไว้

### Checklist ✅
- [ ] ลบ `UseHttpsRedirection` ซ้ำ
- [ ] ลบ DI registration ซ้ำใน `IIQE_BL/Binder.cs`
- [ ] ลบ DI registration ซ้ำใน `IIQE_DL/Binder.cs`
- [ ] ลบ commented-out code ยาวๆ
- [ ] Build สำเร็จ
- [ ] Smoke test ผ่าน
