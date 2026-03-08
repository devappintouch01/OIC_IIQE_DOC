# 🧪 คู่มือแก้ไข Phase 4: Quality & Testing

**ระยะเวลา:** สัปดาห์ที่ 25-36  
**เป้าหมาย:** สร้าง testing framework, บังคับ code quality, วางแผน ORM migration  
**Prerequisite:** Phase 3 (God Class refactoring) ทำไปอย่างน้อยบางส่วน

---

## Task 4.1: สร้าง Unit Test Project

### ขั้นตอนที่ 1: สร้างโปรเจกต์

```powershell
cd d:\Works\OICIIQE
dotnet new xunit -n IIQE_Tests
dotnet sln OIC_IIQE.sln add IIQE_Tests\IIQE_Tests.csproj
```

### ขั้นตอนที่ 2: เพิ่ม References & Packages

```xml
<!-- IIQE_Tests/IIQE_Tests.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework> <!-- ปรับตาม version ของโปรเจกต์ -->
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
    <PackageReference Include="xunit" Version="2.*" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.*" />
    <PackageReference Include="Moq" Version="4.*" />
    <PackageReference Include="FluentAssertions" Version="6.*" />
    <PackageReference Include="coverlet.collector" Version="6.*" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\IIQE_BL\IIQE_BL.csproj" />
    <ProjectReference Include="..\IIQE_DL\IIQE_DL.csproj" />
    <ProjectReference Include="..\IIQE_MODELS\IIQE_MODELS.csproj" />
    <ProjectReference Include="..\IIQE_UTILITY\IIQE_UTILITY.csproj" />
  </ItemGroup>
</Project>
```

### โครงสร้างโฟลเดอร์ Test

```
IIQE_Tests/
├── BL/
│   ├── Frontend/
│   │   ├── CorpExam/
│   │   │   ├── CorpExamSearchBLTests.cs
│   │   │   ├── CorpExamApprovalBLTests.cs
│   │   │   └── CorpExamPaymentBLTests.cs
│   │   ├── ExamRequestBLTests.cs
│   │   └── RegisterBLTests.cs
│   └── Backend/
│       ├── BillPaymentBLTests.cs
│       └── ReceiptBLTests.cs
├── Helpers/
│   ├── TestDataFactory.cs
│   └── DataTableBuilder.cs
└── Extensions/
    ├── ConversionExtensionsTests.cs
    └── DataTableExtensionsTests.cs
```

### ขั้นตอนที่ 3: สร้าง Test Helper

```csharp
// IIQE_Tests/Helpers/DataTableBuilder.cs
using System.Data;

namespace IIQE_Tests.Helpers
{
    /// <summary>
    /// Helper สำหรับสร้าง DataTable ใน unit tests
    /// (เพราะ Repository return DataTable)
    /// </summary>
    public class DataTableBuilder
    {
        private readonly DataTable _dt;

        public DataTableBuilder(params string[] columns)
        {
            _dt = new DataTable();
            foreach (var col in columns)
                _dt.Columns.Add(col);
        }

        public DataTableBuilder AddRow(params object[] values)
        {
            _dt.Rows.Add(values);
            return this;
        }

        public DataTable Build() => _dt;

        // === Predefined builders ===
        public static DataTable EmptyTable() => new DataTable();

        public static DataTable MasterList(params (string value, string text)[] items)
        {
            var builder = new DataTableBuilder("Value", "Text");
            foreach (var (value, text) in items)
                builder.AddRow(value, text);
            return builder.Build();
        }
    }
}
```

### ขั้นตอนที่ 4: เขียน Unit Tests

```csharp
// IIQE_Tests/Extensions/ConversionExtensionsTests.cs
using IIQE_UTILITY.Extensions;
using Xunit;
using FluentAssertions;

namespace IIQE_Tests.Extensions
{
    public class ConversionExtensionsTests
    {
        [Theory]
        [InlineData("123", 123)]
        [InlineData("0", 0)]
        [InlineData("9999999999", 9999999999)]
        public void ToInt64Safe_ValidInput_ReturnsCorrectValue(
            string input, long expected)
        {
            input.ToInt64Safe().Should().Be(expected);
        }

        [Theory]
        [InlineData(null)]
        [InlineData("")]
        [InlineData("abc")]
        [InlineData("12.34")]
        public void ToInt64Safe_InvalidInput_ReturnsZero(string input)
        {
            input.ToInt64Safe().Should().Be(0);
        }
    }
}
```

```csharp
// IIQE_Tests/BL/Frontend/CorpExam/CorpExamSearchBLTests.cs
using Moq;
using Xunit;
using FluentAssertions;
using IIQE_BL.Managers.Frontend.Implementations;
using IIQE_DL.Repositories.Frontend.Interfaces;
using IIQE_MODELS.ViewModels.CorpExamRequest;
using IIQE_Tests.Helpers;
using System.Data;

namespace IIQE_Tests.BL.Frontend.CorpExam
{
    public class CorpExamSearchBLTests
    {
        private readonly Mock<IIQE_ICorpExamRequestRepo> _repoMock;
        // ← ใช้ class ที่ refactor แล้วจาก Phase 3:
        // private readonly CorpExamSearchBL _sut;

        public CorpExamSearchBLTests()
        {
            _repoMock = new Mock<IIQE_ICorpExamRequestRepo>();
        }

        [Fact]
        public void GetMasterSearch_WithThaiLanguage_ReturnsThaiLicenseTypes()
        {
            // Arrange
            var param = new MasterParam { lng = "th", user_id = 1, type = 1 };
            
            _repoMock.Setup(r => r.GetMasterLicenseType("th", 1))
                .Returns(DataTableBuilder.MasterList(
                    ("1#1", "P01 ตัวแทนประกันชีวิต"),
                    ("1#2", "P02 ตัวแทนประกันวินาศภัย")
                ));
            
            _repoMock.Setup(r => r.GetMasterCenter("th"))
                .Returns(DataTableBuilder.MasterList(
                    ("1", "ศูนย์สอบกรุงเทพ")
                ));

            // Act
            // var result = _sut.GetMasterSearch(param);

            // Assert
            // result.LicenseType.Should().HaveCount(2);
            // result.ExamCenter.Should().HaveCount(1);
        }

        [Fact]
        public void SearchResult_WithCorporateRole_CallsCorrectRepo()
        {
            // Arrange
            var search = new CorpExamRequestSearch
            {
                role_type_id = 2,  // CorporateMember
                lng = "th",
                license_regist_type = "0"
            };
            
            _repoMock.Setup(r => r.CorpExamRequestSearch(It.IsAny<CorpExamRequestSearch>()))
                .Returns(DataTableBuilder.EmptyTable());

            // Act
            // var result = _sut.SearchResult(search);

            // Assert
            _repoMock.Verify(r => r.CorpExamRequestSearch(
                It.Is<CorpExamRequestSearch>(s => s.role_type_id == 2)), 
                Times.Once);
        }

        [Fact]
        public void SearchResultPagination_DefaultPagination_Uses10PerPage()
        {
            // Arrange
            var search = new CorpExamRequestSearch
            {
                role_type_id = 2,
                lng = "th",
                license_regist_type = "0"
            };
            var pagination = new Pagination { page_number = 0, page_length = 0 };

            _repoMock.Setup(r => r.CorpExamRequestSearchPagination(
                    It.IsAny<CorpExamRequestSearch>(), 
                    It.IsAny<Pagination>()))
                .Returns(DataTableBuilder.EmptyTable());

            // Act
            // var result = _sut.SearchResultPagination(search, pagination);

            // Assert
            _repoMock.Verify(r => r.CorpExamRequestSearchPagination(
                It.IsAny<CorpExamRequestSearch>(),
                It.Is<Pagination>(p => p.page_number == 1 && p.page_length == 10)),
                Times.Once);
        }
    }
}
```

### วิธีรัน Tests

```powershell
cd d:\Works\OICIIQE
dotnet test IIQE_Tests --logger "console;verbosity=detailed"

# กับ coverage report
dotnet test IIQE_Tests --collect:"XPlat Code Coverage"
```

### Checklist ✅
- [ ] สร้าง IIQE_Tests project
- [ ] เพิ่มเข้า solution
- [ ] สร้าง Test Helpers (DataTableBuilder, TestDataFactory)
- [ ] เขียน tests สำหรับ Extension methods (ConversionExtensions, DataTableExtensions)
- [ ] เขียน tests สำหรับ BL classes ที่ refactor แล้ว
- [ ] Coverage ≥ 40% สำหรับ BL layer

---

## Task 4.2: Integration Tests

### สร้าง Integration Test สำหรับ Critical API Endpoints

```csharp
// IIQE_Tests/Integration/CorpExamRequestApiTests.cs
using Microsoft.AspNetCore.Mvc.Testing;
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Xunit;
using FluentAssertions;

namespace IIQE_Tests.Integration
{
    public class CorpExamRequestApiTests : IClassFixture<WebApplicationFactory<IIQE_API.Startup>>
    {
        private readonly HttpClient _client;

        public CorpExamRequestApiTests(WebApplicationFactory<IIQE_API.Startup> factory)
        {
            _client = factory.CreateClient();
            _client.DefaultRequestHeaders.Add("IIQE-Key", "test-api-key");
        }

        [Fact]
        public async Task GetCorpExamRequest_WithoutApiKey_Returns401()
        {
            // Arrange
            var client = new WebApplicationFactory<IIQE_API.Startup>()
                .CreateClient();
            // ไม่ใส่ API Key

            // Act
            var response = await client.GetAsync("/api/CorpExamRequest/search");

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
        }

        [Fact]
        public async Task GetCorpExamRequest_WithApiKey_Returns200()
        {
            // Act
            var response = await _client.GetAsync("/api/CorpExamRequest/search");

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.OK);
        }
    }
}
```

### Critical Flows ที่ต้องมี Integration Tests

| # | Flow | Priority |
|---|---|---|
| 1 | ค้นหาคำขอสมัครสอบ (Search) | 🔴 |
| 2 | สร้าง/แก้ไข/ลบคำขอสมัครสอบ (CRUD) | 🔴 |
| 3 | Submit → Approve → Complete flow | 🔴 |
| 4 | Login / Authentication | 🔴 |
| 5 | Bill Payment creation | 🟠 |
| 6 | Receipt generation | 🟠 |
| 7 | File Upload/Download | 🟡 |

### Checklist ✅
- [ ] สร้าง Integration test project structure
- [ ] Test API Authentication middleware
- [ ] Test critical search endpoints
- [ ] Test CRUD operations
- [ ] Test approval flow (end-to-end)

---

## Task 4.3: Code Analyzer Setup

### เพิ่ม StyleCop + SonarAnalyzer

```xml
<!-- เพิ่มใน Directory.Build.props (สร้างที่ root ของ solution) -->
<Project>
  <ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.2.*">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
    <PackageReference Include="SonarAnalyzer.CSharp" Version="9.*">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

### สร้าง .editorconfig

```ini
# d:\Works\OICIIQE\.editorconfig
root = true

[*.cs]
# Indentation
indent_style = space
indent_size = 4

# Naming conventions
dotnet_naming_rule.private_fields.severity = suggestion
dotnet_naming_rule.private_fields.symbols = private_fields
dotnet_naming_rule.private_fields.style = _camelCase

dotnet_naming_symbols.private_fields.applicable_kinds = field
dotnet_naming_symbols.private_fields.applicable_accessibilities = private

dotnet_naming_style._camelCase.required_prefix = _
dotnet_naming_style._camelCase.capitalization = camel_case

# StyleCop rules — ปิดบาง rule ที่เข้มงวดเกินไปสำหรับโปรเจกต์เดิม
dotnet_diagnostic.SA1101.severity = none  # ไม่บังคับ this.
dotnet_diagnostic.SA1200.severity = none  # using placement
dotnet_diagnostic.SA1633.severity = none  # file header
dotnet_diagnostic.SA1600.severity = suggestion  # XML comments (เริ่มจาก suggestion)

# SonarAnalyzer — เปิด rules สำคัญ
dotnet_diagnostic.S1481.severity = warning  # unused variables
dotnet_diagnostic.S2737.severity = warning  # empty catch
dotnet_diagnostic.S3776.severity = warning  # cognitive complexity
```

### Checklist ✅
- [ ] สร้าง `Directory.Build.props` ที่ root
- [ ] สร้าง `.editorconfig`
- [ ] Build สำเร็จ — ดู warning count
- [ ] ตั้ง baseline — บันทึก warning count ปัจจุบัน
- [ ] ตั้งเป้า — ลด warnings ลง 10% ต่อ sprint

---

## Task 4.4: Evaluate ORM Migration

### เปรียบเทียบ Options

| เกณฑ์ | Raw SQL (ปัจจุบัน) | Dapper | EF Core |
|---|---|---|---|
| **Learning curve** | - | ต่ำ (ใกล้ raw SQL) | สูง |
| **Performance** | สูงสุด | ใกล้เคียง raw SQL | ปานกลาง |
| **Effort to migrate** | - | 🟡 ปานกลาง | 🔴 สูงมาก |
| **Type safety** | ❌ ไม่มี | ✅ มี | ✅ มี |
| **Oracle support** | ✅ | ✅ (ผ่าน Oracle package) | ⚠️ จำกัด |
| **Thread safety** | ❌ (ปัจจุบัน) | ✅ | ✅ |

### คำแนะนำ: ใช้ Dapper

**เหตุผล:**
1. Migration effort ต่ำกว่า EF Core มาก — เพราะ SQL ที่มียังคงใช้ได้
2. Performance ใกล้เคียง raw SQL
3. Oracle support ดีกว่า EF Core
4. แก้ปัญหา thread safety โดยอัตโนมัติ (ใช้ connection per query)

### Migration Strategy — ทีละ Repository

```csharp
// ก่อน (Raw SQL + DataTable)
public DataTable GetMasterLicenseType(string lng)
{
    SQL = "SELECT id, name_th FROM mt_t_license_type WHERE is_active = 1";
    dt = conn.ExecuteReaderWithParams(SQL, _configuration, null);
    return dt;  // ← return DataTable (weak typing)
}

// หลัง (Dapper — return strongly-typed)
public async Task<IEnumerable<MasterModel>> GetMasterLicenseTypeAsync(string lng)
{
    var sql = lng == "en"
        ? "SELECT TO_CHAR(id) AS Value, name_en AS Text FROM mt_t_license_type WHERE is_active = 1"
        : "SELECT TO_CHAR(id) AS Value, name_th AS Text FROM mt_t_license_type WHERE is_active = 1";

    using var connection = new OracleConnection(_connectionString);
    return await connection.QueryAsync<MasterModel>(sql);
}
```

### Checklist ✅
- [ ] ทำ PoC — migrate 1 Repository ง่ายๆ ไป Dapper
- [ ] Benchmark — เปรียบเทียบ performance
- [ ] ตัดสินใจ — Dapper vs EF Core vs อยู่กับ raw SQL
- [ ] สร้าง migration plan — ทีละ Repository
- [ ] (ระยะยาว) Migrate Repository ทีละตัว

---

## Task 4.5: เพิ่มเติม — Health Checks

```csharp
// IIQE_API/Startup.cs
services.AddHealthChecks()
    .AddOracle(Configuration.GetConnectionString("DefaultConnection"), 
        name: "oracle-db",
        tags: new[] { "db" });

// ใน Configure
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
    endpoints.MapHealthChecks("/health");
});
```

---

## Task 4.6: เพิ่มเติม — Structured Logging (Serilog)

```csharp
// Program.cs — เปลี่ยนจาก ClassLogfile มาใช้ Serilog ตรงๆ
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((context, config) =>
        {
            config
                .ReadFrom.Configuration(context.Configuration)
                .Enrich.FromLogContext()
                .WriteTo.Console()
                .WriteTo.File(
                    path: context.Configuration["Logging:FilePath"] + "/log_.txt",
                    rollingInterval: RollingInterval.Day,
                    fileSizeLimitBytes: 10_485_760,
                    retainedFileCountLimit: 30);
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

---

## สรุป Checklist ทั้ง Phase 4

- [ ] สร้าง IIQE_Tests project + เพิ่มเข้า solution
- [ ] เขียน Unit Tests ≥ 40% coverage สำหรับ BL
- [ ] เขียน Integration Tests สำหรับ critical flows
- [ ] ติดตั้ง StyleCop + SonarAnalyzer
- [ ] สร้าง .editorconfig
- [ ] Evaluate ORM — ทำ PoC กับ Dapper
- [ ] เพิ่ม Health Check endpoint
- [ ] ปรับปรุง Logging เป็น Serilog structured logging

> [!TIP]
> **เมื่อจบ Phase 4 ทั้งระบบจะมี:**
> - ✅ Automated tests ที่ catch regression
> - ✅ Code quality enforcement อัตโนมัติ
> - ✅ Health monitoring endpoint
> - ✅ Structured logging ที่ค้นหาได้
> - ✅ แผน ORM migration สำหรับอนาคต
