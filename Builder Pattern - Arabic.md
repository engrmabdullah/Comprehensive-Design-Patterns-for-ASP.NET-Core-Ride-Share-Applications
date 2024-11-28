### Builder Pattern 

----------

#### **1️⃣ يعني إيه Builder Pattern؟**

يا صديقي، تخيل إنك بتطلب عربية من شركة تصنيع، ممكن تقولهم "عايز عربية بأربع كراسي، وسقف بانوراما، وموتور 2000 سي سي." مش لازم تعرف التفاصيل الدقيقة عن طريقة تركيب العربية. الشركة عندها مهندس مسؤول عن بناء العربية خطوة بخطوة، وفي الآخر يسلمك المنتج النهائي.

ده بالظبط اللي بيعمله **Builder Pattern**:

-   بيفصل عملية إنشاء الكائن (Object) عن تفاصيل بنائه.
-   بيسهل إنشاء كائنات معقدة بخطوات متتابعة ومحددة.

----------

#### **2️⃣ المشكلة قبل استخدام Builder Pattern**

في تطبيق **Ride-Share**، لما تيجي تبني كائن زي "رحلة" (Ride) عنده تفاصيل كتيرة:

1.  السائق (Driver).
2.  المستخدم (User).
3.  نقطة البداية والنهاية (Start & End Points).
4.  المسافة (Distance).
5.  السعر (Price).

لو حاولت تعمل كل ده في كود واحد، هيبقى معقد وصعب القراءة والصيانة.

----------

#### 🚫 **الكود قبل Builder Pattern:**

```csharp
public class Ride
{
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public string StartPoint { get; set; }
    public string EndPoint { get; set; }
    public double Distance { get; set; }
    public double Price { get; set; }

    public Ride(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        DriverId = driverId;
        UserId = userId;
        StartPoint = startPoint;
        EndPoint = endPoint;
        Distance = distance;
        Price = price;
    }
}

```

----------

#### **استخدام الكائن في RideService:**

```csharp
var ride = new Ride(1, 2, "Point A", "Point B", 15.5, 30.0);

```

**المشاكل:**

1.  لو عدد الخصائص زاد، الكود هيبقى معقد أكتر.
2.  لو فيه قيمة افتراضية لبعض الخصائص، هتحتاج تعمل أكتر من Constructor.
3.  الكود صعب القراءة ومش واضح.

----------

### **❌ هيكلة المشروع قبل Builder Pattern:**

```plaintext
RootFolder/
├── Domain/
│   └── Entities/
│       └── Ride.cs
└── Application/
    └── Services/
        └── RideService.cs

```

----------

### **3️⃣ الحل باستخدام Builder Pattern**

----------

### **✅ هيكلة المشروع بعد Builder Pattern:**

```plaintext
RootFolder/
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
│   └── Builders/
│       └── RideBuilder.cs
└── Application/
    └── Services/
        └── RideService.cs

```

----------

### **4️⃣ خطوات تنفيذ Builder Pattern**

----------

#### **الخطوة 1: إنشاء كائن Ride بدون تعقيد**

```csharp
public class Ride
{
    public int DriverId { get; private set; }
    public int UserId { get; private set; }
    public string StartPoint { get; private set; }
    public string EndPoint { get; private set; }
    public double Distance { get; private set; }
    public double Price { get; private set; }

    public Ride(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        DriverId = driverId;
        UserId = userId;
        StartPoint = startPoint;
        EndPoint = endPoint;
        Distance = distance;
        Price = price;
    }
}

```

----------

#### **الخطوة 2: إنشاء Builder للكائن Ride**

في مجلد `Domain/Builders`:

```csharp
public class RideBuilder
{
    private int _driverId;
    private int _userId;
    private string _startPoint;
    private string _endPoint;
    private double _distance;
    private double _price;

    public RideBuilder SetDriver(int driverId)
    {
        _driverId = driverId;
        return this;
    }

    public RideBuilder SetUser(int userId)
    {
        _userId = userId;
        return this;
    }

    public RideBuilder SetStartPoint(string startPoint)
    {
        _startPoint = startPoint;
        return this;
    }

    public RideBuilder SetEndPoint(string endPoint)
    {
        _endPoint = endPoint;
        return this;
    }

    public RideBuilder SetDistance(double distance)
    {
        _distance = distance;
        return this;
    }

    public RideBuilder SetPrice(double price)
    {
        _price = price;
        return this;
    }

    public Ride Build()
    {
        return new Ride(_driverId, _userId, _startPoint, _endPoint, _distance, _price);
    }
}

```

**الشرح:**

1.  `RideBuilder` هو كائن مسؤول عن بناء كائن `Ride`.
2.  كل خاصية عندها دالة `Set` لتعيين القيمة المطلوبة.
3.  دالة `Build` تنشئ الكائن النهائي.

----------

#### **الخطوة 3: استخدام RideBuilder في RideService**

في مجلد `Application/Services`:

```csharp
public class RideService
{
    public Ride CreateRide(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        var rideBuilder = new RideBuilder();

        var ride = rideBuilder
            .SetDriver(driverId)
            .SetUser(userId)
            .SetStartPoint(startPoint)
            .SetEndPoint(endPoint)
            .SetDistance(distance)
            .SetPrice(price)
            .Build();

        return ride;
    }
}

```

**الشرح:**

-   `RideBuilder` يُستخدم هنا لتحديد الخصائص المطلوبة وإنشاء الكائن النهائي بطريقة واضحة ومنظمة.

----------

#### **الخطوة 4: إنشاء Controller**

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

    [HttpPost("create")]
    public IActionResult CreateRide([FromBody] CreateRideDto dto)
    {
        var ride = _rideService.CreateRide(dto.DriverId, dto.UserId, dto.StartPoint, dto.EndPoint, dto.Distance, dto.Price);
        return Ok(ride);
    }
}

```

----------

#### **الخطوة 5: تسجيل الخدمات في `Program.cs`**

```csharp
builder.Services.AddScoped<RideService>();

```

----------

### **5️⃣ تأثير Builder Pattern على الكود**

#### **قبل:**

-   الكود كان معقد وصعب الفهم بسبب وجود Constructors متعددة.
-   تعديل خصائص الكائن كان يتطلب إعادة بناء الكود بالكامل.

#### **بعد:**

-   الكود أصبح منظم وواضح باستخدام خطوات محددة لبناء الكائن.
-   يمكنك بسهولة إضافة خصائص جديدة دون التأثير على الكود الحالي.
-   تحسين القراءة والصيانة.

----------

