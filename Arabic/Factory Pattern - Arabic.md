### Factory Pattern 

----------

#### **إيه هو Factory Pattern؟**

يا صديقي، تخيل معايا إنك بتدير تطبيق فيه أنواع مختلفة من الرحلات زي **Standard** و**Luxury** و**Shared**. بدل ما كل شوية تقعد تكتب كود مخصوص عشان تكريت نوع معين، الـ **Factory Pattern** بيعمل لك ورشة جاهزة. الورشة دي هي اللي بتطلعلك النوع اللي انت عايزه بمنتهى السهولة ومن غير ما تدخل في التفاصيل.

----------

### **❌ المشكلة: الطريقة التقليدية بتكرر الكود وبتبقى صعبة الصيانة**

لما تيجي تكريت الرحلات بأنواعها، غالبًا بتضطر تستخدم أكواد زي `if` أو `switch` عشان تحدد نوع الرحلة اللي محتاجها. ده ممكن يسبب:

1.  **تكرار الكود:** نفس فكرة إنشاء الكائنات بتتكرر كل مرة.
2.  **صعوبة الصيانة:** لو عايز تضيف نوع جديد، لازم تعدل في كل مكان فيه الكود ده.
3.  **عدم المرونة:** الكود بيبقى معقد وكل مرة عايز تغيير صغير تبقى لازم تفتح الكود كله.

----------

#### 🛑 **كود بدون Factory Pattern:**

```csharp
public class RideService
{
    public IRide GetRide(string rideType)
    {
        if (rideType == "Standard")
        {
            return new StandardRide();
        }
        else if (rideType == "Luxury")
        {
            return new LuxuryRide();
        }
        else if (rideType == "Shared")
        {
            return new SharedRide();
        }
        else
        {
            throw new ArgumentException("نوع الرحلة غير صحيح!");
        }
    }
}

```

-   **المشكلة هنا؟**  
    كل مرة تحب تضيف نوع جديد زي "Economy" لازم ترجع تغير في الكود ده، وده ممكن يسبب أخطاء أو تكرار كبير.

----------

### **✅ الحل: استخدام Factory Pattern**

الـ **Factory Pattern** بيخليك تحط منطق إنشاء الكائنات في مكان واحد (المصنع) بدل ما تعيد نفس الكود كل مرة. لما تحب تضيف نوع جديد، تروح تضيفه في مكان واحد بس.

----------

### **هيكلة المشروع بعد تطبيق Factory Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideFactory.cs
│   ├── Services/
│   │   └── RideService.cs
├── Domain/
│   ├── Entities/
│   │   ├── IRide.cs
│   │   ├── StandardRide.cs
│   │   ├── LuxuryRide.cs
│   │   └── SharedRide.cs
├── Infrastructure/
│   ├── Factories/
│   │   └── RideFactory.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ خطوات تطبيق Factory Pattern**

#### **الخطوة 1: تعريف الواجهة العامة للرحلات**

دي الواجهة اللي كل أنواع الرحلات هتشتغل من خلالها.

```csharp
public interface IRide
{
    string GetRideDetails();
}

```

----------

#### **الخطوة 2: إنشاء الأنواع المختلفة من الرحلات**

```csharp
public class StandardRide : IRide
{
    public string GetRideDetails()
    {
        return "Standard Ride";
    }
}

public class LuxuryRide : IRide
{
    public string GetRideDetails()
    {
        return "Luxury Ride";
    }
}

public class SharedRide : IRide
{
    public string GetRideDetails()
    {
        return "Shared Ride";
    }
}

```

----------

#### **الخطوة 3: إنشاء واجهة المصنع (Factory Interface)**

```csharp
public interface IRideFactory
{
    IRide CreateRide(string rideType);
}

```

----------

#### **الخطوة 4: تنفيذ المصنع (Factory Implementation)**

هنا كل منطق إنشاء الكائنات هيبقى في مكان واحد.

```csharp
public class RideFactory : IRideFactory
{
    public IRide CreateRide(string rideType)
    {
        return rideType switch
        {
            "Standard" => new StandardRide(),
            "Luxury" => new LuxuryRide(),
            "Shared" => new SharedRide(),
            _ => throw new ArgumentException("نوع الرحلة غير صحيح!"),
        };
    }
}

```

----------

#### **الخطوة 5: استخدام المصنع في الخدمة**

```csharp
public class RideService
{
    private readonly IRideFactory _rideFactory;

    public RideService(IRideFactory rideFactory)
    {
        _rideFactory = rideFactory;
    }

    public IRide GetRide(string rideType)
    {
        return _rideFactory.CreateRide(rideType);
    }
}

```

----------

#### **الخطوة 6: إنشاء Controller**

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

    [HttpGet("{rideType}")]
    public IActionResult GetRideDetails(string rideType)
    {
        var ride = _rideService.GetRide(rideType);
        return Ok(ride.GetRideDetails());
    }
}

```

----------

#### **الخطوة 7: التسجيل في Program.cs**

```csharp
builder.Services.AddScoped<IRideFactory, RideFactory>();
builder.Services.AddScoped<RideService>();

```

----------

### **8️⃣ التأثير قبل وبعد استخدام Factory Pattern**

#### **قبل التحسين:**

-   الكود كان متكرر وكل مرة تحب تضيف حاجة لازم تعدل في كل مكان.
-   صعب تضيف أنواع جديدة بسهولة.
-   صعوبة في صيانة الكود.

#### **بعد التحسين:**

-   كل منطق إنشاء الكائنات بقى في مكان واحد.
-   إضافة أنواع جديدة بقت سهلة ومش بتأثر على الكود القديم.
-   الكود بقى مرتب وأسهل في الصيانة.


