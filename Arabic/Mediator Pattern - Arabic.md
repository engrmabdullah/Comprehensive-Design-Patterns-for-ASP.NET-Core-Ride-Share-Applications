### Mediator Pattern 

----------

#### **1️⃣ يعني إيه Mediator Pattern؟**

يا صديقي، تخيل معايا لو عندك شوية ناس في غرفة وكل واحد بيكلم التاني على طول بشكل مباشر. الدنيا هتبقى زحمة جدًا، وكل واحد هيفضل معتمد على اللي قدامه. لكن لو عندك منظم، زي "وسيط"، كل الناس تكلمه وهو اللي يوصل الرسائل بينهم، الأمور هتكون أسهل وأبسط.

ده بالظبط اللي بيعمله **Mediator Pattern** في الكود. بدل ما كل كائن (Object) يتكلم مع كائن تاني بشكل مباشر، بنضيف كائن وسيط هو اللي يدير الاتصالات دي. وده بيقلل التعقيد ويخلي الكود مرن وسهل الصيانة.

----------

### **2️⃣ المشكلة قبل استخدام Mediator Pattern**

في تطبيق Ride-Share، لما تعمل رحلة جديدة، ممكن يحصل أكتر من حاجة:

1.  تبعت إشعار للسواق.
2.  تسجل بيانات الرحلة في قاعدة البيانات.
3.  تبعت تأكيد للمستخدم.

لو خليت الخدمة اللي بتعمل الرحلة (RideService) هي المسؤولة عن كل ده، هيبقى عندك مشاكل:

1.  **كود معقد:** كل حاجة متشابكة.
2.  **صعوبة التعديل:** لو غيرت طريقة الإشعار أو حفظ البيانات، هتعدل في كذا مكان.
3.  **الاختبار صعب:** لأن كل حاجة مربوطة ببعضها.

----------

#### 🚫 **الكود قبل Mediator Pattern:**

```csharp
public class RideService
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideService()
    {
        _notificationService = new NotificationService();
        _databaseService = new DatabaseService();
    }

    public void CreateRide(Ride ride)
    {
        _databaseService.SaveRide(ride);
        _notificationService.NotifyDriver(ride.DriverId, ride);
        _notificationService.NotifyUser(ride.UserId, ride);
    }
}

```

----------

### **❌ هيكلة المشروع قبل Mediator Pattern:**

```plaintext
RootFolder/
├── Services/
│   ├── RideService.cs
│   ├── NotificationService.cs
│   └── DatabaseService.cs

```

----------

### **3️⃣ الحل باستخدام Mediator Pattern**

دلوقتي، هنعمل كائن "وسيط" هو اللي يدير الاتصالات بين الكائنات المختلفة.

----------

### **✅ هيكلة المشروع بعد Mediator Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideRequestHandler.cs
│   ├── Mediators/
│   │   └── RideMediator.cs
│   ├── Notifications/
│   │   └── RideCreatedNotification.cs
│   ├── Commands/
│   │   └── CreateRideCommand.cs
│   ├── Services/
│   │   └── RideService.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Services/
│   │   ├── NotificationService.cs
│   │   └── DatabaseService.cs

```

----------

### **4️⃣ خطوات تنفيذ Mediator Pattern**

----------

#### **الخطوة 1: إنشاء Command (طلب)**

زي ما تكون بتقول للوسيط: "أعمل الرحلة دي."  
في مجلد `Application/Commands`:

```csharp
public class CreateRideCommand
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
}

```

----------

#### **الخطوة 2: تعريف Notification (إشعار)**

"بلغ السواق والمستخدم بالرحلة."  
في مجلد `Application/Notifications`:

```csharp
public class RideCreatedNotification
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
}

```

----------

#### **الخطوة 3: إنشاء Mediator (الوسيط)**

الكائن الوسيط اللي يدير الطلبات والإشعارات.  
في مجلد `Application/Mediators`:

```csharp
public class RideMediator
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideMediator(NotificationService notificationService, DatabaseService databaseService)
    {
        _notificationService = notificationService;
        _databaseService = databaseService;
    }

    public void Handle(CreateRideCommand command)
    {
        // حفظ البيانات
        _databaseService.SaveRide(new Ride
        {
            Id = command.RideId,
            DriverId = command.DriverId,
            UserId = command.UserId,
            Distance = command.Distance
        });

        // إرسال الإشعارات
        _notificationService.NotifyDriver(command.DriverId, command);
        _notificationService.NotifyUser(command.UserId, command);
    }
}

```

----------

#### **الخطوة 4: تنفيذ الخدمات**

**NotificationService:**  
زي خدمة بتبعت رسائل.

```csharp
public class NotificationService
{
    public void NotifyDriver(int driverId, CreateRideCommand command)
    {
        Console.WriteLine($"Driver {driverId} notified for Ride {command.RideId}");
    }

    public void NotifyUser(int userId, CreateRideCommand command)
    {
        Console.WriteLine($"User {userId} notified for Ride {command.RideId}");
    }
}

```

**DatabaseService:**  
لحفظ البيانات.

```csharp
public class DatabaseService
{
    public void SaveRide(Ride ride)
    {
        Console.WriteLine($"Ride {ride.Id} saved to the database.");
    }
}

```

----------

#### **الخطوة 5: استخدام الوسيط في RideService**

في مجلد `Application/Services`:

```csharp
public class RideService
{
    private readonly RideMediator _rideMediator;

    public RideService(RideMediator rideMediator)
    {
        _rideMediator = rideMediator;
    }

    public void CreateRide(CreateRideCommand command)
    {
        _rideMediator.Handle(command);
    }
}

```

----------

#### **الخطوة 6: إنشاء Controller**

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
    public IActionResult CreateRide([FromBody] CreateRideCommand command)
    {
        _rideService.CreateRide(command);
        return Ok("Ride created successfully!");
    }
}

```

----------

#### **الخطوة 7: تسجيل Mediator والخدمات في `Program.cs`**

```csharp
builder.Services.AddScoped<NotificationService>();
builder.Services.AddScoped<DatabaseService>();
builder.Services.AddScoped<RideMediator>();
builder.Services.AddScoped<RideService>();

```

----------

### **5️⃣ تأثير Mediator Pattern على الكود**

#### **قبل Mediator Pattern:**

-   الخدمات كانت مترابطة مع بعضها.
-   الكود معقد وصعب الصيانة.
-   أي تغيير في منطق خدمة يتطلب تعديل أكواد متعددة.

#### **بعد Mediator Pattern:**

-   الخدمات بتتكلم مع بعضها عن طريق الوسيط.
-   الكود منظم وسهل التعديل.
-   تقليل الترابط وخطأ التغيير.

----------

