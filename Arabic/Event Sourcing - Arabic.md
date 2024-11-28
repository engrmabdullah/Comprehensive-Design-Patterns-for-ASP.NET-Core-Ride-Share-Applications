### Event Sourcing 

----------

#### **1️⃣ يعني إيه Event Sourcing؟**

بص يا صديقي، **Event Sourcing** هو نمط تصميم بيعتمد على تسجيل الأحداث (Events) بدلاً من تحديث البيانات مباشرة في قاعدة البيانات.

-   بدل ما تحفظ الحالة النهائية لأي كيان، بتحفظ كل حدث حصل عليه.
-   الأحداث بتتم بشكل تسلسلي، ومن خلالها تقدر تعيد بناء الحالة النهائية.

----------

#### **2️⃣ مشكلة قبل استخدام Event Sourcing**

في تطبيق Ride-Share، نفترض عندنا طلب ركوب:

-   المستخدم يطلب رحلة.
-   السائق يقبل الرحلة.
-   الرحلة تبدأ.
-   الرحلة تنتهي.

**المشكلة في الطرق التقليدية:**

1.  كل تحديث للحالة بيعدل البيانات بشكل مباشر في قاعدة البيانات.
2.  لو حصلت مشكلة (زي ضياع البيانات)، مش هتقدر تعرف تاريخ الأحداث.
3.  صعوبة في تتبع الحالة الزمنية للرحلات (مين عدّل ومتى؟).
4.  ما فيش سجل كامل للأحداث اللي حصلت.

----------

#### 🚫 **الكود التقليدي (بدون Event Sourcing):**

```csharp
public class RideRequest
{
    public int Id { get; set; }
    public string Status { get; set; } // Pending, Accepted, InProgress, Completed
    public DateTime LastUpdated { get; set; }
}

```

-   أي عملية بتعدل الحالة مباشرة:

```csharp
public void UpdateRideStatus(int rideId, string status)
{
    var ride = _dbContext.Rides.FirstOrDefault(r => r.Id == rideId);
    if (ride != null)
    {
        ride.Status = status;
        ride.LastUpdated = DateTime.UtcNow;
        _dbContext.SaveChanges();
    }
}

```

**المشاكل:**

1.  ما فيش تتبع لكل حدث حصل.
2.  صعوبة تحليل البيانات القديمة.
3.  لو حصلت مشكلة في التعديل، البيانات القديمة بتضيع.

----------

### **❌ هيكلة المشروع قبل Event Sourcing**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── RideRequest.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **3️⃣ الحل باستخدام Event Sourcing**

----------

#### **الفكرة:**

-   بدل ما تحفظ الحالة النهائية بس، تسجل كل حدث (Event) حصل على الكيان.
-   الأحداث بتكون مرتبة زمنيًا.
-   الحالة النهائية لأي كيان بتتبني عن طريق "إعادة تشغيل" كل الأحداث.

----------

### **✅ هيكلة المشروع بعد Event Sourcing**

```plaintext
RootFolder/
├── Application/
│   ├── Events/
│   │   ├── RideRequestedEvent.cs
│   │   ├── RideAcceptedEvent.cs
│   │   ├── RideStartedEvent.cs
│   │   └── RideCompletedEvent.cs
│   ├── Services/
│   │   └── EventStoreService.cs
│   ├── Models/
│   │   └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ خطوات تنفيذ Event Sourcing**

----------

#### **الخطوة 1: تعريف الأحداث (Events)**

##### **مثال: حدث طلب الرحلة**

```csharp
public class RideRequestedEvent
{
    public int RideId { get; set; }
    public int UserId { get; set; }
    public DateTime Timestamp { get; set; }
}

```

##### **مثال: حدث قبول الرحلة**

```csharp
public class RideAcceptedEvent
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public DateTime Timestamp { get; set; }
}

```

----------

#### **الخطوة 2: إنشاء Event Store لتخزين الأحداث**

```csharp
public class EventStoreService
{
    private readonly List<object> _events = new List<object>();

    public void SaveEvent(object @event)
    {
        _events.Add(@event);
    }

    public IEnumerable<object> GetEvents()
    {
        return _events;
    }
}

```

**الشرح:**

-   `SaveEvent` لتسجيل الأحداث.
-   `GetEvents` لاسترجاع كل الأحداث.

----------

#### **الخطوة 3: إنشاء Ride Aggregate لإعادة بناء الحالة**

```csharp
public class Ride
{
    public int Id { get; private set; }
    public string Status { get; private set; }

    public void Apply(RideRequestedEvent @event)
    {
        Id = @event.RideId;
        Status = "Requested";
    }

    public void Apply(RideAcceptedEvent @event)
    {
        Status = "Accepted";
    }

    public void Apply(RideStartedEvent @event)
    {
        Status = "InProgress";
    }

    public void Apply(RideCompletedEvent @event)
    {
        Status = "Completed";
    }
}

```

**الشرح:**

-   الكيان `Ride` بيمثل الحالة النهائية، لكن بيتم تحديثه عن طريق الأحداث.

----------

#### **الخطوة 4: تعديل الـ Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly EventStoreService _eventStore;

    public RideController(EventStoreService eventStore)
    {
        _eventStore = eventStore;
    }

    [HttpPost("request")]
    public IActionResult RequestRide([FromBody] RideRequestedEvent rideRequested)
    {
        _eventStore.SaveEvent(rideRequested);
        return Ok("Ride requested.");
    }

    [HttpPost("accept")]
    public IActionResult AcceptRide([FromBody] RideAcceptedEvent rideAccepted)
    {
        _eventStore.SaveEvent(rideAccepted);
        return Ok("Ride accepted.");
    }

    [HttpGet("rebuild/{rideId}")]
    public IActionResult RebuildRide(int rideId)
    {
        var events = _eventStore.GetEvents();
        var ride = new Ride();
        foreach (var @event in events)
        {
            if (@event is RideRequestedEvent r && r.RideId == rideId) ride.Apply(r);
            if (@event is RideAcceptedEvent a && a.RideId == rideId) ride.Apply(a);
        }
        return Ok(ride);
    }
}

```

----------

#### **الخطوة 5: تسجيل Event Store في `Program.cs`**

```csharp
builder.Services.AddSingleton<EventStoreService>();

```

----------

### **5️⃣ فوائد Event Sourcing**

#### **قبل:**

-   البيانات كانت تُعدل مباشرة، مما يؤدي لفقدان التاريخ.
-   صعوبة تتبع الأحداث الزمنية.

#### **بعد:**

-   كل حدث مسجل بشكل زمني.
-   القدرة على إعادة بناء الحالة في أي وقت.
-   تحسين الشفافية وسهولة تتبع الأخطاء.

----------

