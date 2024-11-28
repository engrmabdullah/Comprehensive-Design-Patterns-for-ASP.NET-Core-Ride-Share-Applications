###  Decorator Pattern 

----------

#### **1️⃣ يعني إيه Decorator Pattern؟**

يا صديقي، تخيل إنك بتشتري كوباية شاي من الكافيه.

-   في الأول بتطلب الشاي العادي.
-   بعدين بتضيف عليه سكر، لبن، أو أي حاجة تانية.

كل إضافة بتحسن الكوباية بطريقة مخصصة من غير ما نعدل في الشاي الأساسي.  
ده بالظبط اللي بيعمله **Decorator Pattern**:

-   بيتيح لك تضيف وظائف جديدة للكائن الأساسي بشكل ديناميكي من غير ما تغير في الكود الأصلي.

----------

#### **2️⃣ المشكلة قبل استخدام Decorator Pattern**

في تطبيق Ride-Share، لو عندنا خدمة حساب تكلفة الرحلة (`RideCostCalculator`) اللي بتحسب التكلفة الأساسية للرحلة بناءً على المسافة:

عايز تضيف حاجات زي:

1.  **رسوم انتظار (Waiting Time Fee).**
2.  **رسوم إضافية لعدد الركاب (Passenger Count Fee).**
3.  **خصومات (Discounts).**

**المشكلة:**  
لو حاولت تضيف كل الوظائف دي في كود واحد، هيبقى معقد ومليان شروط.

----------

#### 🚫 **الكود قبل Decorator Pattern:**

```csharp
public class RideCostCalculator
{
    public double CalculateBaseCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        double cost = distance * 2.5; // التكلفة الأساسية
        cost += waitingTime * 0.5; // رسوم الانتظار
        cost += passengerCount > 1 ? (passengerCount - 1) * 1.0 : 0; // رسوم إضافية للركاب
        cost -= discount; // الخصم
        return cost;
    }
}

```

**المشاكل:**

1.  الكود غير مرن؛ كل مرة تضيف وظيفة جديدة هتعدل في الدالة.
2.  الكود لا يتبع **Single Responsibility Principle (SRP)**.
3.  صعب قراءة الكود وصيانته.

----------

### **❌ هيكلة المشروع قبل Decorator Pattern:**

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

### **3️⃣ الحل باستخدام Decorator Pattern**

----------

#### **الفكرة:**

-   هنقسم كل وظيفة إضافية (مثل رسوم الانتظار أو الخصم) إلى **Decorator مستقل**.
-   الديكوريتر ده هيتعامل مع الكائن الأساسي (`RideCostCalculator`) ويضيف وظيفته الخاصة.

----------

### **✅ هيكلة المشروع بعد Decorator Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostCalculator.cs
│   ├── Services/
│   │   ├── BaseCostCalculator.cs
│   │   ├── WaitingTimeFeeDecorator.cs
│   │   ├── PassengerFeeDecorator.cs
│   │   └── DiscountDecorator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ خطوات تنفيذ Decorator Pattern**

----------

#### **الخطوة 1: إنشاء واجهة (Interface) لحساب التكلفة**

في مجلد `Application/Interfaces`:

```csharp
public interface IRideCostCalculator
{
    double CalculateCost(double distance);
}

```

**الشرح:**

-   `IRideCostCalculator` هي الواجهة المشتركة اللي هيستخدمها كل الديكوريترز والكائن الأساسي.

----------

#### **الخطوة 2: إنشاء الكائن الأساسي**

في مجلد `Application/Services`:

```csharp
public class BaseCostCalculator : IRideCostCalculator
{
    public double CalculateCost(double distance)
    {
        return distance * 2.5; // التكلفة الأساسية لكل كيلومتر
    }
}

```

**الشرح:**

-   `BaseCostCalculator` هو الكائن الأساسي اللي بيحسب التكلفة الأساسية فقط.

----------

#### **الخطوة 3: إنشاء Decorators للإضافات**

----------

**WaitingTimeFeeDecorator:**

```csharp
public class WaitingTimeFeeDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly double _waitingTime;

    public WaitingTimeFeeDecorator(IRideCostCalculator innerCalculator, double waitingTime)
    {
        _innerCalculator = innerCalculator;
        _waitingTime = waitingTime;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        return baseCost + (_waitingTime * 0.5); // إضافة رسوم الانتظار
    }
}

```

----------

**PassengerFeeDecorator:**

```csharp
public class PassengerFeeDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly int _passengerCount;

    public PassengerFeeDecorator(IRideCostCalculator innerCalculator, int passengerCount)
    {
        _innerCalculator = innerCalculator;
        _passengerCount = passengerCount;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        var extraFee = _passengerCount > 1 ? (_passengerCount - 1) * 1.0 : 0;
        return baseCost + extraFee; // إضافة رسوم الركاب الإضافيين
    }
}

```

----------

**DiscountDecorator:**

```csharp
public class DiscountDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly double _discount;

    public DiscountDecorator(IRideCostCalculator innerCalculator, double discount)
    {
        _innerCalculator = innerCalculator;
        _discount = discount;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        return baseCost - _discount; // تطبيق الخصم
    }
}

```

----------

#### **الخطوة 4: استخدام Decorators في RideService**

في مجلد `Application/Services`:

```csharp
public class RideService
{
    public double CalculateRideCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        IRideCostCalculator calculator = new BaseCostCalculator();

        calculator = new WaitingTimeFeeDecorator(calculator, waitingTime);
        calculator = new PassengerFeeDecorator(calculator, passengerCount);
        calculator = new DiscountDecorator(calculator, discount);

        return calculator.CalculateCost(distance);
    }
}

```

**الشرح:**

-   نبدأ بالكائن الأساسي (`BaseCostCalculator`).
-   نضيف كل ديكوريتر خطوة بخطوة حسب الوظائف المطلوبة.

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

    [HttpPost("calculate-cost")]
    public IActionResult CalculateCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        var cost = _rideService.CalculateRideCost(distance, waitingTime, passengerCount, discount);
        return Ok($"Total ride cost: {cost} USD");
    }
}

```

----------

### **5️⃣ تأثير Decorator Pattern على الكود**

#### **قبل:**

-   الكود كان معقد بسبب دمج كل الوظائف في مكان واحد.
-   تعديل أي وظيفة كان يتطلب تعديل الدالة الأساسية.

#### **بعد:**

-   كل وظيفة أصبحت منفصلة في Decorator خاص بها.
-   يمكنك إضافة أو تعديل الوظائف بسهولة دون لمس الكود الحالي.
-   الكود أصبح يتبع مبدأ **Single Responsibility Principle**.

----------


