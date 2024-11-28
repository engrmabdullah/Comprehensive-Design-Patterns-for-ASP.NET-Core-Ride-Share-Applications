###  Proxy Pattern 

----------

#### **1️⃣ ما هو Proxy Pattern؟**

**Proxy Pattern** هو نمط تصميم يسمح بإضافة طبقة وسيطة (Proxy) للتحكم في طريقة الوصول إلى كائن معين. الهدف الرئيسي من هذا النمط هو:

-   تحسين الأداء باستخدام التخزين المؤقت (Caching).
-   التحكم في الوصول إلى الموارد الثقيلة أو الحساسة.
-   إضافة وظائف إضافية دون تغيير الكود الأصلي.

**مثال من الحياة الواقعية:**  
تخيل أنك تزور مكتبة كبيرة للحصول على كتاب. بدلًا من أن تبحث بنفسك، هناك أمين مكتبة (Proxy) يبحث نيابة عنك ويوفر لك الكتاب إذا كان متاحًا.

----------

#### **2️⃣ المشكلة قبل استخدام Proxy Pattern**

في تطبيق Ride-Share، إذا أردنا إنشاء خدمة لحساب تكلفة الرحلة (`RideCostCalculator`):

1.  البيانات يتم جلبها مباشرة من قاعدة البيانات أو API خارجي.
2.  إذا كانت البيانات نفسها تُطلب عدة مرات، فسيؤدي ذلك إلى استهلاك الموارد بشكل زائد.
3.  نحتاج إلى تقليل هذا الحمل باستخدام **Caching**.

----------

#### 🚫 **الكود قبل استخدام Proxy Pattern**

```csharp
public class RideCostCalculator
{
    public double GetCost(int rideId)
    {
        // استعلام من قاعدة البيانات
        Console.WriteLine($"Fetching cost for ride {rideId} from database...");
        return 50.0; // مثال على تكلفة الرحلة
    }
}

```

**المشاكل:**

1.  كل مرة يتم استدعاء نفس البيانات، يتم الاتصال بقاعدة البيانات.
2.  لا يوجد تخزين مؤقت (Caching) للنتائج المتكررة.

----------

### **❌ هيكلة المشروع قبل Proxy Pattern**

```plaintext
RootFolder/
├── Application/
│   └── Services/
│       └── RideCostCalculator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **3️⃣ الحل باستخدام Proxy Pattern**

----------

#### **الفكرة:**

نقوم بإضافة **Proxy** يعمل كطبقة وسيطة بين العميل (`RideService`) والكائن الرئيسي (`RideCostCalculator`).

-   يقوم الـ Proxy بالتحقق مما إذا كانت البيانات المطلوبة موجودة في الكاش (Cache).
-   إذا كانت البيانات موجودة، يتم إرجاعها مباشرة من الكاش.
-   إذا لم تكن موجودة، يتم استدعاء الكائن الرئيسي وجلب البيانات وتخزينها.

----------

### **✅ هيكلة المشروع بعد Proxy Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostCalculator.cs
│   ├── Services/
│   │   ├── RideCostCalculator.cs
│   │   └── RideCostProxy.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ خطوات تنفيذ Proxy Pattern**

----------

#### **الخطوة 1: إنشاء واجهة (Interface)**

في مجلد `Application/Interfaces`:

```csharp
public interface IRideCostCalculator
{
    double GetCost(int rideId);
}

```

**الشرح:**

-   هذه الواجهة تتيح لنا استخدام الكود الأساسي أو الـ Proxy دون تغيير الكود الذي يعتمد عليهما.

----------

#### **الخطوة 2: إنشاء الكود الأساسي**

في مجلد `Application/Services`:

```csharp
public class RideCostCalculator : IRideCostCalculator
{
    public double GetCost(int rideId)
    {
        Console.WriteLine($"Fetching cost for ride {rideId} from database...");
        return 50.0; // تكلفة الرحلة (مثال)
    }
}

```

**الشرح:**

-   `RideCostCalculator` هو الكود الأساسي الذي يتعامل مع قاعدة البيانات مباشرة.

----------

#### **الخطوة 3: إنشاء الـ Proxy**

في نفس المجلد:

```csharp
public class RideCostProxy : IRideCostCalculator
{
    private readonly IRideCostCalculator _calculator;
    private readonly Dictionary<int, double> _cache = new();

    public RideCostProxy(IRideCostCalculator calculator)
    {
        _calculator = calculator;
    }

    public double GetCost(int rideId)
    {
        if (_cache.ContainsKey(rideId))
        {
            Console.WriteLine($"Returning cached cost for ride {rideId}...");
            return _cache[rideId];
        }

        var cost = _calculator.GetCost(rideId);
        _cache[rideId] = cost;
        return cost;
    }
}

```

**الشرح:**

-   `RideCostProxy` يضيف **التخزين المؤقت (Caching)**:
    -   إذا كانت البيانات موجودة في الكاش، يتم إرجاعها مباشرة.
    -   إذا لم تكن موجودة، يتم استدعاء الكود الأساسي وحفظ البيانات في الكاش.

----------

#### **الخطوة 4: استخدام Proxy في RideService**

```csharp
public class RideService
{
    private readonly IRideCostCalculator _costCalculator;

    public RideService(IRideCostCalculator costCalculator)
    {
        _costCalculator = costCalculator;
    }

    public double GetRideCost(int rideId)
    {
        return _costCalculator.GetCost(rideId);
    }
}

```

**الشرح:**

-   `RideService` يعتمد فقط على الواجهة (`IRideCostCalculator`)، مما يتيح له استخدام أي تنفيذ سواء الأساسي أو الـ Proxy.

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

    [HttpGet("cost/{rideId}")]
    public IActionResult GetCost(int rideId)
    {
        var cost = _rideService.GetRideCost(rideId);
        return Ok($"The cost of ride {rideId} is {cost} USD");
    }
}

```

----------

#### **الخطوة 6: تسجيل الخدمات في `Program.cs`**

```csharp
builder.Services.AddScoped<RideCostCalculator>();
builder.Services.AddScoped<IRideCostCalculator, RideCostProxy>();
builder.Services.AddScoped<RideService>();

```

**الشرح:**

-   يتم تسجيل الـ Proxy في النظام بدلاً من الكود الأساسي.

----------

### **5️⃣ تأثير Proxy Pattern على الكود**

#### **قبل:**

-   كل استدعاء كان يؤدي إلى Query جديد.
-   استهلاك كبير للموارد.

#### **بعد:**

-   تم تحسين الأداء باستخدام التخزين المؤقت.
-   تقليل عدد الاستعلامات إلى قاعدة البيانات.
-   الكود أصبح أكثر مرونة وقابلية للتوسيع.

----------

