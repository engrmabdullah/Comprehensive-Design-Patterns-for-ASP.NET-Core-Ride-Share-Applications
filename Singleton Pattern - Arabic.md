

#### **1️⃣ ما هو Singleton Pattern؟**

يا صديقي، **Singleton Pattern** هو نمط تصميم (Design Pattern) بيضمن إن في كائن واحد بس بيتعمل منه نسخة واحدة طول فترة تشغيل التطبيق. الفكرة هنا إنك بدل ما تنشئ الكائن كل مرة، تقدر تحتفظ بنسخة واحدة وتستخدمها في أي مكان في المشروع.

----------

### **2️⃣ المشكلة قبل استخدام Singleton Pattern**

في تطبيقات زي **Ride-Share**، بيكون عندنا حاجات محتاجة تبقى ثابتة ومتوفرة لكل أجزاء التطبيق زي **قائمة السائقين المتاحين** أو **إعدادات التطبيق**. بدون Singleton Pattern، ممكن يحصل:

1.  **استهلاك موارد:** إنشاء نفس الكائن أكتر من مرة.
2.  **عدم تناسق البيانات:** الكائنات المختلفة ممكن تحتوي على بيانات مختلفة.
3.  **صعوبة إدارة:** الكود هيبقى معقد ومليان تكرار.

----------

#### **🚫 الكود قبل تطبيق Singleton Pattern:**

```csharp
public class DriverCache
{
    public List<Driver> AvailableDrivers { get; set; } = new List<Driver>();
}

public class RideService
{
    public void AddDriver(Driver driver)
    {
        var driverCache = new DriverCache(); // يتم إنشاء كائن جديد كل مرة
        driverCache.AvailableDrivers.Add(driver);
    }

    public List<Driver> GetAvailableDrivers()
    {
        var driverCache = new DriverCache(); // كائن جديد يعني البيانات مش متناسقة
        return driverCache.AvailableDrivers;
    }
}

```

-   **المشكلة؟**  
    كل مرة تنادي على `DriverCache`، التطبيق بيعمل كائن جديد. النتيجة إن كل جزء في التطبيق بيشتغل على نسخة مختلفة.

----------

### **❌ هيكلة المشروع قبل تطبيق Singleton Pattern:**

```plaintext
RootFolder/
├── Data/
│   ├── DriverCache.cs
├── Services/
│   └── RideService.cs
├── Entities/
│   └── Driver.cs
└── Program.cs

```

----------

### **3️⃣ الحل: استخدام Singleton Pattern**

مع **Singleton Pattern**، بنخلي كائن **DriverCache** نسخة واحدة لكل التطبيق، بحيث كل أجزاء التطبيق تشارك نفس النسخة. ده بيوفر الموارد ويضمن تناسق البيانات.

----------

### **✅ هيكلة المشروع بعد تطبيق Singleton Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IDriverCache.cs
│   ├── Services/
│   │   └── RideService.cs
├── Domain/
│   ├── Entities/
│   │   └── Driver.cs
├── Infrastructure/
│   ├── Singletons/
│   │   └── DriverCache.cs
├── WebApi/
│   ├── Controllers/
│   │   └── DriverController.cs
└── Program.cs

```

----------

### **4️⃣ خطوات تطبيق Singleton Pattern في Ride-Share**

#### **الخطوة 1: إنشاء Singleton DriverCache**

في مجلد `Infrastructure/Singletons/DriverCache.cs`:

```csharp
public class DriverCache
{
    private static DriverCache _instance;
    private static readonly object _lock = new object();

    public List<Driver> AvailableDrivers { get; set; } = new List<Driver>();

    // منع إنشاء الكائن من الخارج
    private DriverCache() { }

    public static DriverCache Instance
    {
        get
        {
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new DriverCache();
                }
                return _instance;
            }
        }
    }
}

```

----------

#### **الخطوة 2: تعريف كيان Driver**

في مجلد `Domain/Entities/Driver.cs`:

```csharp
public class Driver
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsAvailable { get; set; }
}

```

----------

#### **الخطوة 3: استخدام Singleton في الخدمة**

في مجلد `Application/Services/RideService.cs`:

```csharp
public class RideService
{
    private readonly DriverCache _driverCache;

    public RideService()
    {
        _driverCache = DriverCache.Instance;
    }

    public void AddDriver(Driver driver)
    {
        _driverCache.AvailableDrivers.Add(driver);
    }

    public List<Driver> GetAvailableDrivers()
    {
        return _driverCache.AvailableDrivers;
    }
}

```

----------

#### **الخطوة 4: إنشاء Controller لإدارة السائقين**

في مجلد `WebApi/Controllers/DriverController.cs`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly RideService _rideService;

    public DriverController()
    {
        _rideService = new RideService();
    }

    [HttpPost("add")]
    public IActionResult AddDriver(Driver driver)
    {
        _rideService.AddDriver(driver);
        return Ok("Driver added successfully.");
    }

    [HttpGet("available")]
    public IActionResult GetAvailableDrivers()
    {
        var drivers = _rideService.GetAvailableDrivers();
        return Ok(drivers);
    }
}

```

----------

#### **الخطوة 5: تسجيل Singleton في `Program.cs`**

```csharp
builder.Services.AddSingleton<DriverCache>(DriverCache.Instance);
builder.Services.AddScoped<RideService>();

```

----------

### **5️⃣ التأثير قبل وبعد استخدام Singleton Pattern**

#### **قبل التحسين:**

-   كل استدعاء بيعمل نسخة جديدة من الكائن.
-   البيانات مش متناسقة بين أجزاء التطبيق.
-   استهلاك زائد للموارد.

#### **بعد التحسين:**

-   كائن واحد بيتم استخدامه في كل مكان في التطبيق.
-   البيانات متناسقة.
-   تحسين الأداء وتقليل استهلاك الموارد.

----------
### شرح الكود بالتفصيل

----------

#### **تعريف الكلاس:**

```csharp
public class DriverCache

```

-   هذا كلاس اسمه `DriverCache`، الغرض منه هو تخزين قائمة السائقين المتاحين بطريقة مركزية بحيث يتم الوصول إليها من أي مكان في التطبيق.

----------

#### **تعريف الحقل `_instance`:**

```csharp
private static DriverCache _instance;

```

-   الحقل `_instance` هو متغير من نوع `DriverCache`، يتم تعريفه كـ **static**، مما يعني أنه مرتبط بالكلاس نفسه وليس بأي كائن معين.
-   الهدف منه هو الاحتفاظ بالنسخة الوحيدة (Singleton) التي سيتم استخدامها في التطبيق.

----------

#### **تعريف الحقل `_lock`:**

```csharp
private static readonly object _lock = new object();

```

-   `_lock` هو كائن من نوع `object` يُستخدم لضمان أن عملية الوصول إلى النسخة الوحيدة من الكلاس (Singleton) ستكون **Thread-Safe**.
-   إذا كان التطبيق يعمل في بيئة متعددة الـ Threads (مثل خدمات الويب)، هذا الحقل يمنع إنشاء أكثر من نسخة عند الوصول للكلاس في نفس الوقت.

----------

#### **تعريف قائمة `AvailableDrivers`:**

```csharp
public List<Driver> AvailableDrivers { get; set; } = new List<Driver>();

```

-   `AvailableDrivers` هي خاصية (Property) من نوع قائمة تحتوي على السائقين المتاحين.
-   يتم تعريفها كـ **public**، مما يعني أن أي جزء من الكود يمكنه الوصول إليها لإضافة أو قراءة السائقين.
-   **القيمة الافتراضية:** قائمة فارغة.

----------

#### **تعريف الـ Constructor:**

```csharp
private DriverCache() { }

```

-   **Constructor** خاص بالكلاس، وهو المسؤول عن إنشاء كائن جديد.
-   تم تعريفه كـ **private**، مما يعني أنه لا يمكن لأي جزء خارجي من الكود إنشاء كائن جديد من `DriverCache`.
-   الغرض هو منع إنشاء أكثر من نسخة من الكلاس (Singleton).

----------

#### **تعريف الخاصية `Instance`:**

```csharp
public static DriverCache Instance

```

-   `Instance` هي خاصية من نوع `DriverCache` يتم الوصول إليها عبر الكلاس نفسه وليس عبر كائن معين بسبب كونها **static**.
-   الهدف هو إرجاع النسخة الوحيدة من الكلاس (Singleton) أو إنشاؤها إذا لم تكن موجودة.

----------

#### **داخل `Instance`:**

##### **القفل لضمان الـ Thread-Safety:**

```csharp
lock (_lock)

```

-   **lock** يضمن أنه إذا حاول أكثر من Thread الوصول إلى الخاصية `Instance` في نفس الوقت، فإن واحدًا فقط منهم سيقوم بإنشاء النسخة، والبقية سيستخدمون النسخة التي تم إنشاؤها.

##### **إنشاء النسخة إذا لم تكن موجودة:**

```csharp
if (_instance == null)
{
    _instance = new DriverCache();
}

```

-   **الشرط:** يتم التحقق إذا لم يكن هناك نسخة موجودة بالفعل (أي أن `_instance` تساوي `null`).
-   **الإنشاء:** إذا لم تكن النسخة موجودة، يتم إنشاء كائن جديد من `DriverCache` وتخزينه في `_instance`.

##### **إرجاع النسخة:**

```csharp
return _instance;

```

-   يتم إرجاع النسخة الوحيدة التي تم إنشاؤها أو الموجودة مسبقًا.

----------

### **الهدف من الكود بالكامل:**

-   يضمن أن هناك **نسخة واحدة فقط** من `DriverCache` يتم استخدامها طوال فترة تشغيل التطبيق.
-   يتم ذلك عبر:
    1.  **Constructor خاص:** يمنع إنشاء أكثر من نسخة.
    2.  **خاصية `Instance`:** توفر النسخة الوحيدة بطريقة آمنة مع دعم البيئة متعددة الـ Threads.
    3.  **قائمة السائقين:** توفر تخزين مركزي للسائقين المتاحين لتسهيل إدارتهم في جميع أنحاء التطبيق.
