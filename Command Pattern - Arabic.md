### Command Pattern

----------

#### **1️⃣ يعني إيه Command Pattern؟**

بص يا صديقي، تخيل لو أنت في مطعم وعايز تطلب أكل. بدل ما تكلم الشيف مباشرة وتقول له اعمل كذا وكذا، أنت بتقول للجرسون "عايز بيتزا." الجرسون هنا هو اللي بياخد طلبك وينفذه من غير ما تدخل أنت في تفاصيل طريقة التحضير.

ده بالظبط اللي بيعمله **Command Pattern**.

-   النمط ده بيقسم الطلب (Command) عن التنفيذ (Execution).
-   الفكرة إنك بتبعت أمر أو طلب، والخدمة هي اللي تتعامل مع تفاصيل التنفيذ.

----------

#### **2️⃣ المشكلة قبل استخدام Command Pattern**

في تطبيق **Ride-Share**، لما يكون عندنا عمليات زي:

1.  إنشاء رحلة جديدة.
2.  تحديث بيانات الرحلة.
3.  إلغاء رحلة.

الكود بيتعامل مع الخدمات مباشرة. لو كل عملية متشابكة مع الخدمات التانية، الدنيا بتبقى معقدة وصعبة الصيانة.

----------

#### 🚫 **الكود قبل استخدام Command Pattern:**

```csharp
public class RideService
{
    private readonly RideRepository _rideRepository;

    public RideService(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void CreateRide(Ride ride)
    {
        // حفظ الرحلة في قاعدة البيانات
        _rideRepository.Add(ride);
    }

    public void CancelRide(int rideId)
    {
        var ride = _rideRepository.GetById(rideId);
        ride.Status = "Cancelled";
        _rideRepository.Update(ride);
    }
}

```

----------

### **❌ هيكلة المشروع قبل Command Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideService.cs
├── Repositories/
│   └── RideRepository.cs
└── Program.cs

```

-   **المشكلة:**
    1.  الخدمات متشابكة مع بعضها.
    2.  كل عملية مرتبطة مباشرة بـ `RideRepository`.
    3.  صعوبة في إضافة أو تعديل العمليات الجديدة.

----------

### **3️⃣ الحل باستخدام Command Pattern**

Command Pattern بيخلي كل عملية مستقلة عن التانية. العملية بتتكتب كـ **Command** منفصلة، وبيتم تنفيذها عن طريق **Handler** خاص.

----------

### **✅ هيكلة المشروع بعد Command Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Commands/
│   │   ├── CreateRideCommand.cs
│   │   ├── CancelRideCommand.cs
│   ├── Handlers/
│   │   ├── CreateRideCommandHandler.cs
│   │   ├── CancelRideCommandHandler.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Repositories/
│   │   └── RideRepository.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ خطوات تنفيذ Command Pattern**

----------

#### **الخطوة 1: تعريف Command**

**CreateRideCommand:**  
ده الأمر اللي بيستخدمه المستخدم لإنشاء رحلة.  
في مجلد `Application/Commands`:

```csharp
public class CreateRideCommand
{
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
}

```

**CancelRideCommand:**  
ده الأمر اللي بيستخدمه المستخدم لإلغاء الرحلة.

```csharp
public class CancelRideCommand
{
    public int RideId { get; set; }
}

```

----------

#### **الخطوة 2: إنشاء Handlers**

كل Handler مسؤول عن تنفيذ الأمر المرتبط به.

**CreateRideCommandHandler:**  
في مجلد `Application/Handlers`:

```csharp
public class CreateRideCommandHandler
{
    private readonly RideRepository _rideRepository;

    public CreateRideCommandHandler(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void Handle(CreateRideCommand command)
    {
        var ride = new Ride
        {
            DriverId = command.DriverId,
            UserId = command.UserId,
            Distance = command.Distance,
            Status = "Created"
        };

        _rideRepository.Add(ride);
    }
}

```

**CancelRideCommandHandler:**

```csharp
public class CancelRideCommandHandler
{
    private readonly RideRepository _rideRepository;

    public CancelRideCommandHandler(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void Handle(CancelRideCommand command)
    {
        var ride = _rideRepository.GetById(command.RideId);
        if (ride != null)
        {
            ride.Status = "Cancelled";
            _rideRepository.Update(ride);
        }
    }
}

```

----------

#### **الخطوة 3: إنشاء الكيان (Entity)**

في مجلد `Domain/Entities`:

```csharp
public class Ride
{
    public int Id { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
    public string Status { get; set; }
}

```

----------

#### **الخطوة 4: تنفيذ Repository**

في مجلد `Infrastructure/Repositories`:

```csharp
public class RideRepository
{
    private readonly List<Ride> _rides = new();

    public void Add(Ride ride)
    {
        _rides.Add(ride);
    }

    public Ride GetById(int id)
    {
        return _rides.FirstOrDefault(r => r.Id == id);
    }

    public void Update(Ride ride)
    {
        var existingRide = GetById(ride.Id);
        if (existingRide != null)
        {
            existingRide.Status = ride.Status;
        }
    }
}

```

----------

#### **الخطوة 5: إنشاء Controller**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly CreateRideCommandHandler _createRideHandler;
    private readonly CancelRideCommandHandler _cancelRideHandler;

    public RideController(
        CreateRideCommandHandler createRideHandler,
        CancelRideCommandHandler cancelRideHandler)
    {
        _createRideHandler = createRideHandler;
        _cancelRideHandler = cancelRideHandler;
    }

    [HttpPost("create")]
    public IActionResult CreateRide([FromBody] CreateRideCommand command)
    {
        _createRideHandler.Handle(command);
        return Ok("Ride created successfully!");
    }

    [HttpPost("cancel")]
    public IActionResult CancelRide([FromBody] CancelRideCommand command)
    {
        _cancelRideHandler.Handle(command);
        return Ok("Ride cancelled successfully!");
    }
}

```

----------

#### **الخطوة 6: تسجيل الخدمات في Program.cs**

```csharp
builder.Services.AddScoped<RideRepository>();
builder.Services.AddScoped<CreateRideCommandHandler>();
builder.Services.AddScoped<CancelRideCommandHandler>();

```

----------

### **5️⃣ تأثير Command Pattern على الكود**

#### **قبل:**

-   كل العمليات كانت مكتوبة في `RideService`.
-   الكود معقد وغير منظم.
-   صعوبة في إضافة أو تعديل العمليات الجديدة.

#### **بعد:**

-   كل عملية أصبحت مستقلة في `Command` خاص بها.
-   الكود منظم وسهل الصيانة.
-   إضافة أي عملية جديدة لا يؤثر على الكود الحالي.

----------


