### Dependency Injection 

----------

#### **1️⃣ ما هو Dependency Injection؟**

**Dependency Injection** أو **حقن التبعية** هو نمط تصميم (Design Pattern) يتم استخدامه لفصل الكود الأساسي عن التبعيات التي يحتاجها.

-   الفكرة هي "حقن" الكائنات المطلوبة (Dependencies) بدلًا من إنشائها يدويًا، مما يعزز مرونة الكود وسهولة اختباره.

----------

#### **2️⃣ المشكلة قبل استخدام Dependency Injection**

في تطبيق مثل **Ride-Share**، الخدمات تعتمد على تبعيات مثل:

1.  الوصول إلى قاعدة البيانات.
2.  استخدام API خارجي.
3.  الاعتماد على خدمات أخرى مثل حساب التكلفة.

إذا أنشأنا هذه التبعيات مباشرة داخل الخدمات:

1.  **صعوبة الصيانة:** يجب تعديل الكود في كل مرة تتغير طريقة عمل التبعية.
2.  **تعقيد الاختبار:** الكود يعتمد على كائنات حقيقية مثل قواعد البيانات، مما يجعل الاختبارات أبطأ وأكثر تعقيدًا.
3.  **خرق مبدأ المسؤولية الواحدة (SRP):** الخدمة نفسها مسؤولة عن إنشاء التبعيات وإدارتها.

----------

#### 🚫 **الكود قبل استخدام Dependency Injection (Code Without Pattern):**

```csharp
public class RideCostService
{
    private readonly RideDatabase _rideDatabase;

    public RideCostService()
    {
        _rideDatabase = new RideDatabase(); // يتم إنشاء التبعية داخليًا
    }

    public double CalculateRideCost(int rideId)
    {
        var ride = _rideDatabase.GetRideById(rideId);
        return ride.Distance * 1.5;
    }
}

```

-   **مشاكل هذا الكود:**
    1.  لا يمكن استبدال `RideDatabase` أثناء الاختبار.
    2.  إذا تغيرت طريقة الحصول على البيانات، يجب تعديل الكود هنا.
    3.  الكود غير مرن ومتشابك.

----------

### **❌ هيكلة المشروع قبل تطبيق Dependency Injection:**

```plaintext
RootFolder/
├── Services/
│   └── RideCostService.cs
├── Data/
│   └── RideDatabase.cs
└── Program.cs

```

----------

### **3️⃣ الحل: استخدام Dependency Injection**

باستخدام **Dependency Injection**، يتم حقن التبعيات في الخدمات بدلاً من إنشائها داخل الكود.  
هذا يؤدي إلى فصل واضح بين المسؤوليات ويجعل الكود أكثر مرونة وقابلية للاختبار.

----------

### **✅ هيكلة المشروع بعد تطبيق Dependency Injection:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideDatabase.cs
│   ├── Services/
│   │   └── RideCostService.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Data/
│   │   └── RideDatabase.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ خطوات تطبيق Dependency Injection**

----------

#### **الخطوة 1: تعريف واجهة (Interface) للتبعية**

في مجلد `Application/Interfaces`:

```csharp
public interface IRideDatabase
{
    Ride GetRideById(int rideId);
}

```

----------

#### **الخطوة 2: تنفيذ التبعية في كلاس منفصل**

في مجلد `Infrastructure/Data`:

```csharp
public class RideDatabase : IRideDatabase
{
    public Ride GetRideById(int rideId)
    {
        // محاكاة الحصول على بيانات الرحلة من قاعدة البيانات
        return new Ride { Id = rideId, Distance = 10.0 };
    }
}

```

----------

#### **الخطوة 3: استخدام التبعية في الخدمة**

في مجلد `Application/Services`:

```csharp
public class RideCostService
{
    private readonly IRideDatabase _rideDatabase;

    // يتم حقن التبعية هنا
    public RideCostService(IRideDatabase rideDatabase)
    {
        _rideDatabase = rideDatabase;
    }

    public double CalculateRideCost(int rideId)
    {
        var ride = _rideDatabase.GetRideById(rideId);
        return ride.Distance * 1.5;
    }
}

```

----------

#### **الخطوة 4: إنشاء كيان Ride**

في مجلد `Domain/Entities`:

```csharp
public class Ride
{
    public int Id { get; set; }
    public double Distance { get; set; }
}

```

----------

#### **الخطوة 5: إنشاء Controller لاستخدام الخدمة**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly RideCostService _rideCostService;

    public RideController(RideCostService rideCostService)
    {
        _rideCostService = rideCostService;
    }

    [HttpGet("cost/{rideId}")]
    public IActionResult GetRideCost(int rideId)
    {
        var cost = _rideCostService.CalculateRideCost(rideId);
        return Ok(cost);
    }
}

```

----------

#### **الخطوة 6: تسجيل التبعيات في `Program.cs`**

```csharp
builder.Services.AddScoped<IRideDatabase, RideDatabase>();
builder.Services.AddScoped<RideCostService>();

```

----------

### **5️⃣ كيف تغلبنا على الصعوبات؟**

#### **1. صعوبة الاختبار بسبب الاعتماد على كائنات حقيقية**

-   باستخدام واجهة (`IRideDatabase`)، يمكننا استبدال الكائن الحقيقي بـ **Mock** أو **Stub** أثناء الاختبار.

##### **مثال على اختبار باستخدام Mock:**

```csharp
var mockRideDatabase = new Mock<IRideDatabase>();
mockRideDatabase.Setup(db => db.GetRideById(It.IsAny<int>()))
                .Returns(new Ride { Id = 1, Distance = 20.0 });

var service = new RideCostService(mockRideDatabase.Object);

var cost = service.CalculateRideCost(1);

Assert.Equal(30.0, cost); // 20 * 1.5

```

----------

#### **2. صعوبة الصيانة عند تغيير التبعيات**

-   عند تغيير طريقة الحصول على البيانات (مثل استبدال `RideDatabase` بخدمة API)، كل ما عليك فعله هو تنفيذ واجهة جديدة، ولا تحتاج إلى تعديل الكود الأساسي.

##### **إضافة خدمة جديدة:**

```csharp
public class RideApiDatabase : IRideDatabase
{
    public Ride GetRideById(int rideId)
    {
        // استدعاء API للحصول على بيانات الرحلة
        return new Ride { Id = rideId, Distance = 15.0 };
    }
}

```

ثم تسجيلها في `Program.cs`:

```csharp
builder.Services.AddScoped<IRideDatabase, RideApiDatabase>();

```

----------

#### **3. تحسين الأداء والكود النظيف**

-   الخدمات الآن تعتمد على الواجهات فقط، مما يجعل الكود:
    -   أكثر تنظيماً.
    -   سهل الفهم.
    -   متوافقًا مع مبادئ SOLID.

----------

### **6️⃣ التأثير قبل وبعد استخدام Dependency Injection**

#### **قبل:**

-   الكود معقد وغير مرن.
-   صعوبة في الاختبار بسبب الاعتماد على كائنات حقيقية.
-   الكود غير قابل للصيانة بسهولة.

#### **بعد:**

-   الكود أصبح مرنًا ويمكن استبدال التبعيات بسهولة.
-   يمكن استخدام Mocks أو Stubs في الاختبارات.
-   أصبح الكود متوافقًا مع مبدأ المسؤولية الواحدة (SRP).

----------

