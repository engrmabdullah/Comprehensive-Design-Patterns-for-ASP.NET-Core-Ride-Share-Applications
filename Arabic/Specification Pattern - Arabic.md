### Specification Pattern

----------

#### **1️⃣ يعني إيه Specification Pattern؟**

بص يا صديقي، تخيل إنك شغال في شركة توصيل وبتحتاج تعمل فلترة للسواقين المتاحين.

-   عايز تختار السواقين اللي في نطاق معين.
-   السواقين اللي عندهم تقييم عالي.
-   اللي عربياتهم جديدة، وهكذا.

بدل ما تعمل الشروط دي جوه الكود الأساسي، **Specification Pattern** بيقولك:

-   طلع كل الشروط (Specifications) في كائنات منفصلة.
-   بعدين تقدر تجمعهم مع بعض (Combine) بسهولة.

----------

#### **2️⃣ المشكلة قبل استخدام Specification Pattern**

في تطبيق Ride-Share، عندنا خدمة عايزين نستخدمها علشان نفلتر السواقين المتاحين بناءً على شروط مختلفة زي:

1.  السواقين اللي عندهم تقييم فوق 4.
2.  السواقين في نطاق معين.
3.  السواقين اللي عربياتهم متوفرة.

**المشكلة:**

1.  الكود هيبقى مليان شروط `if` و`where`.
2.  صعب تضيف شروط جديدة بدون ما تعدل الكود القديم.
3.  صعوبة في إعادة استخدام الشروط.

----------

#### 🚫 **الكود قبل Specification Pattern:**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> GetAvailableDrivers(double userLat, double userLong, double radius, double minRating)
    {
        return _drivers.Where(d =>
            d.IsAvailable &&
            d.Rating >= minRating &&
            CalculateDistance(userLat, userLong, d.Lat, d.Long) <= radius
        ).ToList();
    }

    private double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        var R = 6371; // نصف قطر الأرض بالكيلومترات
        var dLat = (lat2 - lat1) * (Math.PI / 180);
        var dLon = (lon2 - lon1) * (Math.PI / 180);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(lat1 * (Math.PI / 180)) * Math.Cos(lat2 * (Math.PI / 180)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
}

```

**المشاكل:**

1.  الكود مليان شروط صعبة التعديل.
2.  صعوبة في إضافة شروط جديدة.
3.  إعادة استخدام الشروط في أماكن تانية مستحيلة.

----------

### **❌ هيكلة المشروع قبل Specification Pattern**

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

### **3️⃣ الحل باستخدام Specification Pattern**

----------

#### **الفكرة:**

-   كل شرط من الشروط (مثل "متاح" أو "في النطاق") يكون في كائن مستقل (Specification).
-   تقدر تجمع الشروط باستخدام عمليات مثل `AND` و`OR`.
-   الكود الأساسي هيبقى بسيط جدًا وممكن تعيد استخدام الشروط بسهولة.

----------

### **✅ هيكلة المشروع بعد Specification Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── ISpecification.cs
│   ├── Specifications/
│   │   ├── BaseSpecification.cs
│   │   ├── AvailableDriverSpecification.cs
│   │   ├── DriverInRadiusSpecification.cs
│   │   └── HighRatingDriverSpecification.cs
│   ├── Services/
│   │   └── DriverService.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **4️⃣ خطوات تنفيذ Specification Pattern**

----------

#### **الخطوة 1: تعريف واجهة Specification**

في مجلد `Application/Interfaces`:

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
}

```

**الشرح:**

-   `ISpecification` بتعرف وظيفة واحدة وهي `IsSatisfiedBy` اللي بتحدد إذا كان الكائن يطابق الشرط ولا لأ.

----------

#### **الخطوة 2: إنشاء Specification أساسي**

في مجلد `Application/Specifications`:

```csharp
public abstract class BaseSpecification<T> : ISpecification<T>
{
    public abstract bool IsSatisfiedBy(T entity);

    public BaseSpecification<T> And(BaseSpecification<T> other)
    {
        return new AndSpecification<T>(this, other);
    }
}

```

----------

#### **الخطوة 3: إنشاء شروط محددة**

##### شرط السائق المتاح:

```csharp
public class AvailableDriverSpecification : BaseSpecification<Driver>
{
    public override bool IsSatisfiedBy(Driver driver)
    {
        return driver.IsAvailable;
    }
}

```

##### شرط السائق في نطاق معين:

```csharp
public class DriverInRadiusSpecification : BaseSpecification<Driver>
{
    private readonly double _userLat;
    private readonly double _userLong;
    private readonly double _radius;

    public DriverInRadiusSpecification(double userLat, double userLong, double radius)
    {
        _userLat = userLat;
        _userLong = userLong;
        _radius = radius;
    }

    public override bool IsSatisfiedBy(Driver driver)
    {
        var distance = CalculateDistance(_userLat, _userLong, driver.Lat, driver.Long);
        return distance <= _radius;
    }

    private double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        // نفس كود حساب المسافة السابق
        var R = 6371; // نصف قطر الأرض
        var dLat = (lat2 - lat1) * (Math.PI / 180);
        var dLon = (lon2 - lon1) * (Math.PI / 180);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(lat1 * (Math.PI / 180)) * Math.Cos(lat2 * (Math.PI / 180)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
}

```

##### شرط السائق بتقييم عالي:

```csharp
public class HighRatingDriverSpecification : BaseSpecification<Driver>
{
    private readonly double _minRating;

    public HighRatingDriverSpecification(double minRating)
    {
        _minRating = minRating;
    }

    public override bool IsSatisfiedBy(Driver driver)
    {
        return driver.Rating >= _minRating;
    }
}

```

----------

#### **الخطوة 4: استخدام الشروط في DriverService**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> GetAvailableDrivers(ISpecification<Driver> specification)
    {
        return _drivers.Where(specification.IsSatisfiedBy).ToList();
    }
}

```

----------

#### **الخطوة 5: إنشاء Controller**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly DriverService _driverService;

    public DriverController(DriverService driverService)
    {
        _driverService = driverService;
    }

    [HttpGet("available")]
    public IActionResult GetAvailableDrivers(double userLat, double userLong, double radius, double minRating)
    {
        var availableSpec = new AvailableDriverSpecification();
        var radiusSpec = new DriverInRadiusSpecification(userLat, userLong, radius);
        var ratingSpec = new HighRatingDriverSpecification(minRating);

        var combinedSpec = availableSpec.And(radiusSpec).And(ratingSpec);

        var drivers = _driverService.GetAvailableDrivers(combinedSpec);
        return Ok(drivers);
    }
}

```

----------

#### **الخطوة 6: التسجيل في `Program.cs`**

```csharp
builder.Services.AddScoped<DriverService>();

```

----------

### **5️⃣ فوائد Specification Pattern**

#### **قبل:**

-   كود معقد مليء بالشروط.
-   صعوبة في إضافة أو تعديل الشروط.

#### **بعد:**

-   الشروط موجودة في كائنات مستقلة، مما يسهل إعادة استخدامها.
-   الكود أصبح بسيطًا وسهل التوسيع.

----------

