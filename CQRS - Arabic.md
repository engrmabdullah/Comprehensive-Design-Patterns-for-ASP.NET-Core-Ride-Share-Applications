### CQRS

----------

#### **1️⃣ يعني إيه CQRS؟**

بص يا صديقي، CQRS (Command Query Responsibility Segregation) هو نمط تصميم بيقولك:

-   **قسّم المسئولية** بين الأوامر (Commands) والاستعلامات (Queries).
-   بدلاً من إنك تجمع كل حاجة في كلاس واحد أو خدمة واحدة، خلي كل حاجة ليها شغلها.

**الفكرة:**

-   **Command:** مسئول عن التعديلات (Write Operations) زي إضافة رحلة جديدة أو تحديث حالة السائق.
-   **Query:** مسئول عن القراءة (Read Operations) زي جلب قائمة بالسائقين المتاحين أو تفاصيل رحلة.

----------

#### **2️⃣ المشكلة قبل استخدام CQRS**

في تطبيق Ride-Share، ممكن يبقى عندنا خدمة واحدة زي `DriverService` بتتعامل مع كل العمليات:

1.  جلب السائقين المتاحين.
2.  تحديث حالة السائق.
3.  إضافة رحلة جديدة.

**المشاكل:**

1.  **تشابك الأكواد:** الكود اللي مسئول عن القراءة مختلط بالكود اللي مسئول عن التعديل.
2.  **صعوبة الصيانة:** كل ما تضيف عملية جديدة، الكلاس بيكبر ويصعب فهمه.
3.  **الأداء:** في بعض الأحيان، الاستعلامات المعقدة ممكن تبطئ التعديلات.

----------

#### 🚫 **الكود قبل CQRS:**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    // قراءة: جلب السائقين المتاحين
    public List<Driver> GetAvailableDrivers()
    {
        return _drivers.Where(d => d.IsAvailable).ToList();
    }

    // تعديل: تحديث حالة السائق
    public void UpdateDriverStatus(int driverId, bool isAvailable)
    {
        var driver = _drivers.FirstOrDefault(d => d.Id == driverId);
        if (driver != null)
        {
            driver.IsAvailable = isAvailable;
        }
    }

    // تعديل: إضافة رحلة جديدة
    public void AddTrip(Trip trip)
    {
        // منطق إضافة رحلة
    }
}

```

**المشاكل:**

1.  كل العمليات (قراءة وتعديل) متشابكة في نفس الكلاس.
2.  لو حصلت مشكلة في استعلام معين، ممكن تأثر على باقي العمليات.

----------

### **❌ هيكلة المشروع قبل CQRS**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── DriverService.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **3️⃣ الحل باستخدام CQRS**

----------

#### **الفكرة:**

-   **قسّم العمليات:**
    
    -   كل العمليات المتعلقة بالقراءة (Queries) تكون في جزء مستقل.
    -   كل العمليات المتعلقة بالتعديل (Commands) تكون في جزء آخر.
-   **ميزة التقسيم:**
    
    -   الكود هيبقى نظيف ومنظم.
    -   كل جزء مسئول عن وظيفة واحدة، مما يسهل الصيانة.

----------

### **✅ هيكلة المشروع بعد CQRS**

```plaintext
RootFolder/
├── Application/
│   ├── Commands/
│   │   ├── AddTripCommand.cs
│   │   ├── UpdateDriverStatusCommand.cs
│   ├── Queries/
│   │   └── GetAvailableDriversQuery.cs
│   ├── Handlers/
│   │   ├── AddTripCommandHandler.cs
│   │   ├── UpdateDriverStatusCommandHandler.cs
│   │   └── GetAvailableDriversQueryHandler.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **4️⃣ خطوات تنفيذ CQRS**

----------

#### **الخطوة 1: إنشاء الأوامر (Commands)**

##### **مثال: تحديث حالة السائق**

```csharp
public class UpdateDriverStatusCommand
{
    public int DriverId { get; set; }
    public bool IsAvailable { get; set; }
}

```

----------

#### **الخطوة 2: إنشاء الاستعلامات (Queries)**

##### **مثال: جلب السائقين المتاحين**

```csharp
public class GetAvailableDriversQuery
{
    // ممكن تضيف شروط هنا لو محتاج
}

```

----------

#### **الخطوة 3: إنشاء المعالجات (Handlers)**

##### **Handler لتحديث حالة السائق:**

```csharp
public class UpdateDriverStatusCommandHandler
{
    private readonly List<Driver> _drivers;

    public UpdateDriverStatusCommandHandler(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public void Handle(UpdateDriverStatusCommand command)
    {
        var driver = _drivers.FirstOrDefault(d => d.Id == command.DriverId);
        if (driver != null)
        {
            driver.IsAvailable = command.IsAvailable;
        }
    }
}

```

##### **Handler لجلب السائقين المتاحين:**

```csharp
public class GetAvailableDriversQueryHandler
{
    private readonly List<Driver> _drivers;

    public GetAvailableDriversQueryHandler(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> Handle(GetAvailableDriversQuery query)
    {
        return _drivers.Where(d => d.IsAvailable).ToList();
    }
}

```

----------

#### **الخطوة 4: تعديل الـ Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly UpdateDriverStatusCommandHandler _updateHandler;
    private readonly GetAvailableDriversQueryHandler _queryHandler;

    public DriverController(
        UpdateDriverStatusCommandHandler updateHandler,
        GetAvailableDriversQueryHandler queryHandler)
    {
        _updateHandler = updateHandler;
        _queryHandler = queryHandler;
    }

    [HttpGet("available")]
    public IActionResult GetAvailableDrivers()
    {
        var query = new GetAvailableDriversQuery();
        var drivers = _queryHandler.Handle(query);
        return Ok(drivers);
    }

    [HttpPost("update-status")]
    public IActionResult UpdateDriverStatus([FromBody] UpdateDriverStatusCommand command)
    {
        _updateHandler.Handle(command);
        return Ok("Driver status updated.");
    }
}

```

----------

#### **الخطوة 5: التسجيل في `Program.cs`**

```csharp
builder.Services.AddScoped<UpdateDriverStatusCommandHandler>();
builder.Services.AddScoped<GetAvailableDriversQueryHandler>();

```

----------

### **5️⃣ فوائد CQRS**

#### **قبل:**

-   كود مشوش يجمع بين القراءة والتعديل.
-   صعوبة في إضافة ميزات جديدة.

#### **بعد:**

-   فصل كامل بين القراءة والتعديل.
-   كود نظيف وسهل الفهم والصيانة.
-   تحسين الأداء، حيث يمكن تحسين القراءة أو التعديل بشكل مستقل.

----------

