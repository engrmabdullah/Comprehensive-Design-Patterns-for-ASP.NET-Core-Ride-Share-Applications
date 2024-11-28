###  Adapter Pattern 

----------

#### **1️⃣ يعني إيه Adapter Pattern؟**

بص يا صديقي، تخيل إنك اشتريت جهاز من أمريكا، ومخرج الكهرباء فيه مختلف عن المخرج في بلدك. علشان تشغل الجهاز، هتحتاج حاجة اسمها محول (Adapter)، اللي بيخلي الجهاز يتوافق مع مصدر الكهرباء.

ده بالظبط اللي بيعمله **Adapter Pattern**.

-   بيخلي كائنات (Objects) بأنواع مختلفة تشتغل مع بعض، حتى لو طرق تعاملهم (Interfaces) مش متوافقة.

----------

#### **2️⃣ المشكلة قبل استخدام Adapter Pattern**

في تطبيق Ride-Share، لنفترض إنك عايز تضيف **خدمة حساب المسافة (Distance Calculation)** باستخدام مكتبتين مختلفتين:

1.  مكتبة بتستخدم نظام الإحداثيات (Lat, Long).
2.  مكتبة تانية بتستخدم نظام الخرائط (Map System).

المشكلة هنا إن كل مكتبة عندها واجهة (Interface) مختلفة.

----------

#### 🚫 **الكود قبل Adapter Pattern:**

```csharp
// مكتبة لحساب المسافة باستخدام Lat/Long
public class LatLongDistanceCalculator
{
    public double Calculate(double lat1, double lon1, double lat2, double lon2)
    {
        // منطق حساب المسافة
        return Math.Sqrt(Math.Pow(lat2 - lat1, 2) + Math.Pow(lon2 - lon1, 2));
    }
}

// مكتبة لحساب المسافة باستخدام الخرائط
public class MapDistanceCalculator
{
    public double GetDistance(string startPoint, string endPoint)
    {
        // منطق حساب المسافة
        return 15.0; // افترضت قيمة
    }
}

// RideService يحاول استخدام المكتبتين
public class RideService
{
    private readonly LatLongDistanceCalculator _latLongCalculator;

    public RideService(LatLongDistanceCalculator latLongCalculator)
    {
        _latLongCalculator = latLongCalculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _latLongCalculator.Calculate(lat1, lon1, lat2, lon2);
    }
}

```

**المشاكل:**

1.  إذا أردت استخدام مكتبة أخرى (MapDistanceCalculator)، ستحتاج تعديل الكود الحالي.
2.  الكود غير مرن وصعب التوسيع.

----------

### **❌ هيكلة المشروع قبل Adapter Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideService.cs
├── Libraries/
│   ├── LatLongDistanceCalculator.cs
│   └── MapDistanceCalculator.cs
└── Program.cs

```

----------

### **3️⃣ الحل باستخدام Adapter Pattern**

----------

#### **الفكرة:**

هننشئ واجهة مشتركة (Interface) لكل خدمات حساب المسافة، وبعدها ننشئ **Adapter** لكل مكتبة، بحيث نتعامل مع نفس الواجهة في الكود الرئيسي.

----------

### **✅ هيكلة المشروع بعد Adapter Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IDistanceCalculator.cs
│   ├── Adapters/
│   │   ├── LatLongAdapter.cs
│   │   └── MapAdapter.cs
├── Libraries/
│   ├── LatLongDistanceCalculator.cs
│   └── MapDistanceCalculator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ خطوات تنفيذ Adapter Pattern**

----------

#### **الخطوة 1: إنشاء واجهة (Interface) مشتركة**

في مجلد `Application/Interfaces`:

```csharp
public interface IDistanceCalculator
{
    double CalculateDistance(double lat1, double lon1, double lat2, double lon2);
}

```

**الشرح:**

-   `IDistanceCalculator` هي الواجهة المشتركة التي ستستخدمها جميع الخدمات.

----------

#### **الخطوة 2: إنشاء Adapters للمكتبات**

**Adapter للمكتبة الأولى (LatLongDistanceCalculator):**

```csharp
public class LatLongAdapter : IDistanceCalculator
{
    private readonly LatLongDistanceCalculator _calculator;

    public LatLongAdapter(LatLongDistanceCalculator calculator)
    {
        _calculator = calculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _calculator.Calculate(lat1, lon1, lat2, lon2);
    }
}

```

**Adapter للمكتبة الثانية (MapDistanceCalculator):**

```csharp
public class MapAdapter : IDistanceCalculator
{
    private readonly MapDistanceCalculator _calculator;

    public MapAdapter(MapDistanceCalculator calculator)
    {
        _calculator = calculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        // تحويل الإحداثيات إلى نصوص
        string startPoint = $"{lat1},{lon1}";
        string endPoint = $"{lat2},{lon2}";
        return _calculator.GetDistance(startPoint, endPoint);
    }
}

```

**الشرح:**

-   كل Adapter يتعامل مع مكتبة مختلفة، لكنه يوفر نفس واجهة `IDistanceCalculator`.

----------

#### **الخطوة 3: استخدام Adapters في RideService**

في مجلد `Application/Services`:

```csharp
public class RideService
{
    private readonly IDistanceCalculator _distanceCalculator;

    public RideService(IDistanceCalculator distanceCalculator)
    {
        _distanceCalculator = distanceCalculator;
    }

    public double GetRideDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _distanceCalculator.CalculateDistance(lat1, lon1, lat2, lon2);
    }
}

```

**الشرح:**

-   `RideService` يتعامل فقط مع الواجهة المشتركة `IDistanceCalculator`، مما يجعل الكود مرنًا لإضافة مكتبات جديدة.

----------

#### **الخطوة 4: تسجيل الخدمات في Program.cs**

```csharp
builder.Services.AddSingleton<LatLongDistanceCalculator>();
builder.Services.AddSingleton<IDistanceCalculator, LatLongAdapter>();
builder.Services.AddScoped<RideService>();

```

**الشرح:**

-   يمكن تبديل `LatLongAdapter` بـ `MapAdapter` بسهولة إذا قررت استخدام مكتبة مختلفة.

----------

#### **الخطوة 5: إنشاء Controller**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly RideService _rideService;

    public RideController(RideService rideService)
    {
        _rideService = rideService;
    }

    [HttpGet("distance")]
    public IActionResult GetDistance(double lat1, double lon1, double lat2, double lon2)
    {
        var distance = _rideService.GetRideDistance(lat1, lon1, lat2, lon2);
        return Ok($"The distance is {distance} km");
    }
}

```

----------

### **5️⃣ تأثير Adapter Pattern على الكود**

#### **قبل:**

-   كل مكتبة تحتاج تعديل مباشر في الخدمة الرئيسية (`RideService`).
-   الكود غير مرن وغير قابل للتوسيع بسهولة.

#### **بعد:**

-   يمكن استخدام أي مكتبة جديدة ببساطة عن طريق إضافة Adapter.
-   `RideService` غير مرتبطة بمكتبة محددة، مما يجعلها أكثر مرونة.
-   يلتزم الكود بمبدأ **Open/Closed Principle** (مفتوح للإضافة ومغلق للتعديل).

----------


