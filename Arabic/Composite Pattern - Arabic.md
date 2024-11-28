### Composite Pattern

----------

#### **1️⃣ يعني إيه Composite Pattern؟**

بص يا صديقي، تخيل إنك بتشغل شركة توصيل فيها عربيات صغيرة وعربيات كبيرة، وكل عربية ممكن تشيل صناديق بأحجام مختلفة.

-   الصناديق الصغيرة ممكن تتحط جوه صناديق أكبر.
-   في النهاية، لازم تعرف الوزن الكلي لكل عربية مهما كانت الصناديق مرتبطة ببعضها.

**Composite Pattern** بيحل المشكلة دي عن طريق:

-   التعامل مع الكائنات الفردية (Individual Objects) والمركّبة (Composite Objects) بنفس الطريقة.

----------

#### **2️⃣ المشكلة قبل استخدام Composite Pattern**

في تطبيق Ride-Share، عندنا خدمة لحساب إجمالي التكلفة أو المسافة المقطوعة لكل الرحلات.

-   بعض الرحلات مرتبطة برحلات فرعية (Nested Trips).
-   الكود لازم يفرق بين الرحلات الفردية (Single Trips) والرحلات المجمعة (Group of Trips).

**المشكلة:**

1.  الكود هيبقى مليان شروط (Conditions) علشان يفرق بين الفردي والمركّب.
2.  صعوبة في إضافة أنواع جديدة من الرحلات.

----------

#### 🚫 **الكود قبل Composite Pattern**

```csharp
public class SingleTrip
{
    public double Distance { get; set; }

    public double GetTotalDistance()
    {
        return Distance;
    }
}

public class GroupTrip
{
    public List<SingleTrip> Trips { get; set; } = new List<SingleTrip>();

    public double GetTotalDistance()
    {
        double totalDistance = 0;
        foreach (var trip in Trips)
        {
            totalDistance += trip.GetTotalDistance();
        }
        return totalDistance;
    }
}

```

**المشاكل:**

1.  `GroupTrip` و `SingleTrip` بيتم التعامل معاهم بشكل منفصل.
2.  لو عايز تضيف نوع جديد من الرحلات، لازم تعدّل كل مكان في الكود.

----------

### **❌ هيكلة المشروع قبل Composite Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   ├── SingleTrip.cs
│   │   └── GroupTrip.cs
└── WebApi/
    └── Controllers/
        └── TripController.cs

```

----------

### **3️⃣ الحل باستخدام Composite Pattern**

----------

#### **الفكرة:**

بدل ما نعامل الرحلات الفردية والجماعية بشكل مختلف:

-   هننشئ واجهة (Interface) مشتركة.
-   كل نوع من الرحلات (الفردية أو الجماعية) هيطبق نفس الواجهة.
-   ممكن نتعامل مع الكائنات الفردية والمركبة بنفس الطريقة.

----------

### **✅ هيكلة المشروع بعد Composite Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── ITripComponent.cs
│   ├── Services/
│   │   ├── SingleTrip.cs
│   │   ├── GroupTrip.cs
└── WebApi/
    └── Controllers/
        └── TripController.cs

```

----------

### **4️⃣ خطوات تنفيذ Composite Pattern**

----------

#### **الخطوة 1: تعريف واجهة مشتركة**

في مجلد `Application/Interfaces`:

```csharp
public interface ITripComponent
{
    double GetTotalDistance();
}

```

**الشرح:**

-   `ITripComponent` هي الواجهة المشتركة لكل أنواع الرحلات.
-   كل نوع من الرحلات هيطبق نفس الوظيفة `GetTotalDistance`.

----------

#### **الخطوة 2: إنشاء الرحلة الفردية**

في مجلد `Application/Services`:

```csharp
public class SingleTrip : ITripComponent
{
    public double Distance { get; set; }

    public double GetTotalDistance()
    {
        return Distance;
    }
}

```

**الشرح:**

-   `SingleTrip` بتمثل رحلة فردية بتنفيذ واجهة `ITripComponent`.

----------

#### **الخطوة 3: إنشاء الرحلة الجماعية**

```csharp
public class GroupTrip : ITripComponent
{
    private readonly List<ITripComponent> _trips = new List<ITripComponent>();

    public void AddTrip(ITripComponent trip)
    {
        _trips.Add(trip);
    }

    public double GetTotalDistance()
    {
        double totalDistance = 0;
        foreach (var trip in _trips)
        {
            totalDistance += trip.GetTotalDistance();
        }
        return totalDistance;
    }
}

```

**الشرح:**

-   `GroupTrip` بتمثل مجموعة رحلات وبتطبق واجهة `ITripComponent`.
-   بتحتوي على قائمة من الكائنات سواء فردية أو جماعية.

----------

#### **الخطوة 4: إنشاء الخدمة `TripService`**

```csharp
public class TripService
{
    public double CalculateTotalDistance(ITripComponent trip)
    {
        return trip.GetTotalDistance();
    }
}

```

**الشرح:**

-   `TripService` بتتعامل مع `ITripComponent` بشكل عام، وده يوفر مرونة كبيرة.

----------

#### **الخطوة 5: إنشاء Controller**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class TripController : ControllerBase
{
    private readonly TripService _tripService;

    public TripController(TripService tripService)
    {
        _tripService = tripService;
    }

    [HttpGet("total-distance")]
    public IActionResult GetTotalDistance()
    {
        var singleTrip = new SingleTrip { Distance = 10 };
        var anotherSingleTrip = new SingleTrip { Distance = 20 };

        var groupTrip = new GroupTrip();
        groupTrip.AddTrip(singleTrip);
        groupTrip.AddTrip(anotherSingleTrip);

        var totalDistance = _tripService.CalculateTotalDistance(groupTrip);
        return Ok($"Total distance: {totalDistance} km");
    }
}

```

----------

#### **الخطوة 6: التسجيل في `Program.cs`**

```csharp
builder.Services.AddScoped<TripService>();

```

----------

### **5️⃣ تأثير Composite Pattern على الكود**

#### **قبل:**

-   التعامل مع الرحلات الفردية والجماعية كان منفصلًا تمامًا.
-   الكود كان مليان شروط للتفرقة بين أنواع الرحلات.

#### **بعد:**

-   كل أنواع الرحلات بتنفذ نفس الواجهة (`ITripComponent`).
-   الكود أصبح مرن جدًا وسهل التوسيع.
-   أي نوع جديد من الرحلات يمكن إضافته بسهولة.

----------


