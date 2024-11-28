### Observer Pattern

----------

#### **1️⃣ ما هو Observer Pattern؟**

Observer Pattern هو نمط تصميم (Design Pattern) يُستخدم لتنظيم العلاقة بين كائنات (Objects) بحيث:

1.  يكون هناك **كائن رئيسي (Subject)** مسؤول عن التحديثات.
2.  يتم إشعار **المراقبين (Observers)** تلقائيًا عندما يحدث تغيير في الكائن الرئيسي.

----------

### **المشكلة قبل Observer Pattern**

#### الكود القديم:

```csharp
public class RideService
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideService(NotificationService notificationService, DatabaseService databaseService)
    {
        _notificationService = notificationService;
        _databaseService = databaseService;
    }

    public void UpdateRideStatus(int rideId, string status)
    {
        // تحديث حالة الرحلة
        _databaseService.UpdateRideStatus(rideId, status);

        // إرسال الإشعارات
        _notificationService.NotifyDriver(rideId, status);
        _notificationService.NotifyUser(rideId, status);
    }
}

```

#### **المشاكل:**

1.  `RideService` مسؤول عن تحديث الحالة وإرسال الإشعارات.
2.  أي إضافة لمراقب جديد (Observer) ستتطلب تعديل الكود.
3.  هذا يُخالف مبدأ **Open/Closed Principle**، حيث يجب أن يكون الكود مغلقًا للتعديل ومفتوحًا للإضافة.

----------

### **الحل باستخدام Observer Pattern**

الآن، سنعيد كتابة الكود باستخدام **Observer Pattern**.

----------

#### **2️⃣ إنشاء الهيكل باستخدام Observer Pattern**

----------

#### **خطوة 1: إنشاء واجهة (Interface) للمراقبين**

```csharp
public interface IObserver
{
    void Update(int rideId, string status);
}

```

**الشرح:**

-   `IObserver` هي واجهة (Interface) تمثل كل مراقب (Observer).
-   الدالة `Update` هي الدالة التي يستدعيها **Subject** لإشعار كل مراقب.

----------

#### **خطوة 2: إنشاء واجهة للكائن الأساسي (Subject)**

```csharp
public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify(int rideId, string status);
}

```

**الشرح:**

1.  `Attach` تضيف مراقب جديد للقائمة.
2.  `Detach` تزيل مراقب من القائمة.
3.  `Notify` تُخبر جميع المراقبين بحدوث تغيير.

----------

#### **خطوة 3: الكائن الأساسي (Subject)**

```csharp
public class RideSubject : ISubject
{
    private readonly List<IObserver> _observers = new();

    public void Attach(IObserver observer)
    {
        _observers.Add(observer);
    }

    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }

    public void Notify(int rideId, string status)
    {
        foreach (var observer in _observers)
        {
            observer.Update(rideId, status);
        }
    }
}

```

**الشرح:**

1.  **قائمة المراقبين:** `_observers` تخزن كل المراقبين المربوطين بـ `RideSubject`.
2.  **إضافة مراقب:** `Attach` تضيف مراقب جديد إلى القائمة.
3.  **إزالة مراقب:** `Detach` تزيل مراقب من القائمة.
4.  **إشعار المراقبين:** `Notify` تمر على كل المراقبين في القائمة وتستدعي `Update` لتحديثهم.

----------

#### **خطوة 4: إنشاء المراقبين (Observers)**

----------

**DriverObserver:**

```csharp
public class DriverObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"Driver notified: Ride {rideId} is now {status}");
    }
}

```

**الشرح:**

-   هذا المراقب يُشعر السائق عندما تتغير حالة الرحلة.
-   يتم عرض رسالة في `Console` عند إشعار السائق.

----------

**UserObserver:**

```csharp
public class UserObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"User notified: Ride {rideId} is now {status}");
    }
}

```

**الشرح:**

-   هذا المراقب يُشعر المستخدم بحالة الرحلة الجديدة.

----------

**LoggingObserver:**

```csharp
public class LoggingObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"Log: Ride {rideId} status changed to {status}");
    }
}

```

**الشرح:**

-   هذا المراقب يسجل التغييرات في حالة الرحلة.

----------

#### **خطوة 5: استخدام الكائن الأساسي (Subject) في الخدمة**

```csharp
public class RideService
{
    private readonly RideSubject _rideSubject;

    public RideService(RideSubject rideSubject)
    {
        _rideSubject = rideSubject;
    }

    public void UpdateRideStatus(int rideId, string status)
    {
        // إشعار جميع المراقبين
        _rideSubject.Notify(rideId, status);
    }
}

```

**الشرح:**

1.  يتم حقن `RideSubject` في `RideService` باستخدام **Dependency Injection**.
2.  عند تغيير حالة الرحلة، يتم استدعاء `Notify` لإشعار كل المراقبين.

----------

#### **خطوة 6: تسجيل الخدمات في `Program.cs`**

```csharp
var rideSubject = new RideSubject();
rideSubject.Attach(new DriverObserver());
rideSubject.Attach(new UserObserver());
rideSubject.Attach(new LoggingObserver());

builder.Services.AddSingleton(rideSubject);
builder.Services.AddScoped<RideService>();

```

**الشرح:**

1.  يتم إنشاء كائن `RideSubject` مرة واحدة (Singleton).
2.  يتم تسجيل المراقبين: `DriverObserver`، `UserObserver`، و`LoggingObserver`.
3.  يتم إضافة `RideService` كخدمة قابلة للحقن.

----------

#### **خطوة 7: إنشاء Controller**

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

    [HttpPost("update-status")]
    public IActionResult UpdateRideStatus(int rideId, string status)
    {
        _rideService.UpdateRideStatus(rideId, status);
        return Ok("Ride status updated successfully!");
    }
}

```

**الشرح:**

1.  يتم استدعاء `RideService` لتحديث حالة الرحلة.
2.  يتم إشعار كل المراقبين بالحالة الجديدة.

----------

### **5️⃣ تأثير Observer Pattern على الكود**

#### **قبل:**

-   الخدمات كانت مترابطة مع بعضها مباشرة.
-   أي تعديل يتطلب تعديل الكود في عدة أماكن.

#### **بعد:**

-   الخدمات أصبحت غير مترابطة.
-   يمكن إضافة مراقبين جدد بسهولة دون تعديل الكود الموجود.

----------

