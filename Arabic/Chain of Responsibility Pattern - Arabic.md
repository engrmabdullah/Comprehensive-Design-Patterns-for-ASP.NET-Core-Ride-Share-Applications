### Chain of Responsibility Pattern 

----------

#### **إيه هو Chain of Responsibility Pattern؟**

يا صديقي، خلينا نقول إنك شغال في تطبيق طلبات رحلات زي أوبر، وعايز تقسم مسؤوليات كل طلب على مراحل معينة. مثلاً:

1.  مرحلة التحقق من البيانات (Validation).
2.  مرحلة التحقق من وجود السائق (Driver Availability).
3.  مرحلة تسعير الرحلة (Pricing Calculation).

بدل ما تعمل كود طويل ومعقد لكل المراحل دي، بنستخدم **Chain of Responsibility Pattern** عشان نفصل كل مرحلة في كلاس مستقل، ونخليها تشتغل بشكل **سلسلة (Chain)**. كل مرحلة في السلسلة تقدر تمرر الطلب للمرحلة اللي بعدها.

----------

#### **المشكلة قبل استخدام النمط**

لو الكود كله متجمع في مكان واحد:

```csharp
public void ProcessRideRequest(RideRequest request)
{
    // تحقق من البيانات
    if (string.IsNullOrEmpty(request.UserId) || string.IsNullOrEmpty(request.Destination))
    {
        Console.WriteLine("Invalid ride request.");
        return;
    }

    // تحقق من وجود السائق
    if (!DriverService.IsDriverAvailable(request))
    {
        Console.WriteLine("No driver available.");
        return;
    }

    // حساب السعر
    request.Price = PricingService.CalculatePrice(request);
    Console.WriteLine($"Ride request processed. Price: {request.Price}");
}

```

**المشكلة:**

-   الكود كبير ومعقد.
-   صعب التعديل أو الإضافة لمراحل جديدة.
-   غير قابل لإعادة الاستخدام.

----------

#### **بعد استخدام Chain of Responsibility Pattern**

#### **1. تقسيم المشروع (Root Folder Structure)**

```
RideShareApp/
├── Domain/
│   ├── Entities/
│   │   └── RideRequest.cs
│   └── Interfaces/
│       └── IRequestHandler.cs
├── Application/
│   └── Handlers/
│       ├── ValidationHandler.cs
│       ├── DriverAvailabilityHandler.cs
│       └── PricingHandler.cs
├── Infrastructure/
├── WebApi/
└── Program.cs

```

----------

#### **2. الكود باستخدام Chain of Responsibility**

##### **الخطوة 1: تعريف RideRequest**

```csharp
public class RideRequest
{
    public string UserId { get; set; }
    public string Destination { get; set; }
    public decimal Price { get; set; }
}

```

----------

##### **الخطوة 2: تعريف IRequestHandler**

```csharp
public interface IRequestHandler
{
    void SetNext(IRequestHandler nextHandler);
    void Handle(RideRequest request);
}

```

----------

##### **الخطوة 3: تنفيذ Handlers**

###### **1. ValidationHandler**

```csharp
public class ValidationHandler : IRequestHandler
{
    private IRequestHandler _nextHandler;

    public void SetNext(IRequestHandler nextHandler)
    {
        _nextHandler = nextHandler;
    }

    public void Handle(RideRequest request)
    {
        if (string.IsNullOrEmpty(request.UserId) || string.IsNullOrEmpty(request.Destination))
        {
            Console.WriteLine("Validation failed: Missing user or destination.");
            return;
        }

        Console.WriteLine("Validation passed.");
        _nextHandler?.Handle(request);
    }
}

```

###### **2. DriverAvailabilityHandler**

```csharp
public class DriverAvailabilityHandler : IRequestHandler
{
    private IRequestHandler _nextHandler;

    public void SetNext(IRequestHandler nextHandler)
    {
        _nextHandler = nextHandler;
    }

    public void Handle(RideRequest request)
    {
        if (!DriverService.IsDriverAvailable(request))
        {
            Console.WriteLine("No driver available.");
            return;
        }

        Console.WriteLine("Driver is available.");
        _nextHandler?.Handle(request);
    }
}

```

###### **3. PricingHandler**

```csharp
public class PricingHandler : IRequestHandler
{
    public void SetNext(IRequestHandler nextHandler)
    {
        // آخر مرحلة في السلسلة
    }

    public void Handle(RideRequest request)
    {
        request.Price = PricingService.CalculatePrice(request);
        Console.WriteLine($"Ride request processed. Price: {request.Price}");
    }
}

```

----------

##### **الخطوة 4: تنفيذ السلسلة في Main**

```csharp
class Program
{
    static void Main(string[] args)
    {
        var rideRequest = new RideRequest
        {
            UserId = "123",
            Destination = "Downtown"
        };

        // إنشاء Handlers وربطهم مع بعض
        var validationHandler = new ValidationHandler();
        var driverAvailabilityHandler = new DriverAvailabilityHandler();
        var pricingHandler = new PricingHandler();

        validationHandler.SetNext(driverAvailabilityHandler);
        driverAvailabilityHandler.SetNext(pricingHandler);

        // بدء السلسلة
        validationHandler.Handle(rideRequest);
    }
}
}

```

----------

#### **3. النتائج**

عند تشغيل الكود:

```plaintext
Validation passed.
Driver is available.
Ride request processed. Price: 25.50

```

----------

#### **4. ليه النمط ده مفيد؟**

1.  **تقسيم المسؤوليات**: كل مرحلة منفصلة، وسهل تعديلها أو استبدالها.
2.  **إضافة مراحل جديدة بسهولة**: لو عايز تضيف مرحلة (مثلاً تأكيد الدفع)، تقدر تضيف Handler جديد.
3.  **قابلية إعادة الاستخدام**: ممكن تعيد استخدام كل Handler في سياقات مختلفة.

----------

