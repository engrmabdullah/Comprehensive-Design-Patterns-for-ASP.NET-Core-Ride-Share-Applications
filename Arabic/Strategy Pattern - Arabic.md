
### Strategy Pattern

----------

#### **1️⃣ ما هو Strategy Pattern؟**

**Strategy Pattern** هو نمط تصميم (Design Pattern) يسمح لك بتحديد مجموعة من الخوارزميات أو الاستراتيجيات المختلفة، وجعل الكود يستخدم أي واحدة منها دون الحاجة لتغيير الكود الأساسي.

-   الغرض هو توفير **مرونة** لاختيار وتنفيذ الخوارزميات المختلفة في وقت التشغيل.

**لماذا نستخدمه؟**

1.  **إزالة الكود المتكرر:** بدلاً من وضع كل الشروط والخوارزميات في كود واحد.
2.  **تسهيل إضافة استراتيجيات جديدة:** إضافة خوارزمية جديدة دون التأثير على الكود الحالي.
3.  **تطبيق مبدأ المسؤولية الواحدة (Single Responsibility Principle):** كل خوارزمية منفصلة في كودها الخاص.

----------

### **2️⃣ المشكلة قبل استخدام Strategy Pattern**

في تطبيق مثل **Ride-Share**، قد تكون هناك طرق متعددة لحساب تكلفة الرحلة. مثلاً:

1.  تكلفة حسب **المسافة فقط**.
2.  تكلفة حسب **المسافة والزمن**.
3.  تكلفة حسب **نوع السيارة**.

لو وضعت كل هذه الاستراتيجيات داخل كود واحد، سيصبح:

1.  **معقدًا ومليئًا بالشروط.**
2.  **صعب الصيانة.**
3.  **غير مرن لإضافة استراتيجيات جديدة.**

----------

#### 🚫 **الكود قبل تطبيق Strategy Pattern (Code Without Pattern):**

```csharp
public class RideCostService
{
    public double CalculateCost(string rideType, double distance, double time)
    {
        if (rideType == "Standard")
        {
            return distance * 1.5; // تكلفة الرحلة القياسية
        }
        else if (rideType == "Luxury")
        {
            return (distance * 3) + (time * 2); // تكلفة الرحلة الفاخرة
        }
        else if (rideType == "Shared")
        {
            return distance * 0.8; // تكلفة الرحلة المشتركة
        }
        else
        {
            throw new ArgumentException("نوع الرحلة غير معروف!");
        }
    }
}

```

-   **المشكلة هنا:**
    1.  الكود مليء بالشروط (`if/else`).
    2.  كل مرة تضيف نوع جديد من الرحلات، تحتاج لتعديل الكود.
    3.  الكود غير قابل للتوسعة بسهولة.

----------

### **❌ هيكلة المشروع قبل تطبيق Strategy Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideCostService.cs
└── Program.cs

```

----------

### **3️⃣ الحل: استخدام Strategy Pattern**

مع **Strategy Pattern**، نضع كل خوارزمية (طريقة حساب التكلفة) في كلاس منفصل، ونستخدم واجهة مشتركة (Interface) لتوحيد طريقة التنفيذ.  
الكود الرئيسي سيختار الاستراتيجية المناسبة بناءً على نوع الرحلة.

----------

### **✅ هيكلة المشروع بعد تطبيق Strategy Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostStrategy.cs
│   ├── Services/
│   │   └── RideCostService.cs
├── Domain/
│   ├── Strategies/
│   │   ├── StandardRideCostStrategy.cs
│   │   ├── LuxuryRideCostStrategy.cs
│   │   └── SharedRideCostStrategy.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ خطوات تطبيق Strategy Pattern**

#### **الخطوة 1: تعريف واجهة الاستراتيجية**

في مجلد `Application/Interfaces`:

```csharp
public interface IRideCostStrategy
{
    double CalculateCost(double distance, double time);
}

```

----------

#### **الخطوة 2: تنفيذ الاستراتيجيات المختلفة**

**Standard Ride Cost:**

```csharp
public class StandardRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return distance * 1.5;
    }
}

```

**Luxury Ride Cost:**

```csharp
public class LuxuryRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return (distance * 3) + (time * 2);
    }
}

```

**Shared Ride Cost:**

```csharp
public class SharedRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return distance * 0.8;
    }
}

```

----------

#### **الخطوة 3: إنشاء خدمة اختيار وتنفيذ الاستراتيجية**

في مجلد `Application/Services`:

```csharp
public class RideCostService
{
    private readonly IRideCostStrategy _rideCostStrategy;

    public RideCostService(IRideCostStrategy rideCostStrategy)
    {
        _rideCostStrategy = rideCostStrategy;
    }

    public double CalculateRideCost(double distance, double time)
    {
        return _rideCostStrategy.CalculateCost(distance, time);
    }
}

```

----------

#### **الخطوة 4: إنشاء Controller لاستخدام الخدمة**

في مجلد `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly IServiceProvider _serviceProvider;

    public RideController(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    [HttpGet("cost")]
    public IActionResult GetRideCost(string rideType, double distance, double time)
    {
        IRideCostStrategy strategy = rideType switch
        {
            "Standard" => _serviceProvider.GetService<StandardRideCostStrategy>(),
            "Luxury" => _serviceProvider.GetService<LuxuryRideCostStrategy>(),
            "Shared" => _serviceProvider.GetService<SharedRideCostStrategy>(),
            _ => throw new ArgumentException("Invalid ride type")
        };

        var rideCostService = new RideCostService(strategy);
        var cost = rideCostService.CalculateRideCost(distance, time);

        return Ok(cost);
    }
}

```

----------

#### **الخطوة 5: تسجيل الاستراتيجيات في `Program.cs`**

```csharp
builder.Services.AddScoped<StandardRideCostStrategy>();
builder.Services.AddScoped<LuxuryRideCostStrategy>();
builder.Services.AddScoped<SharedRideCostStrategy>();

```

----------

### **5️⃣ التأثير قبل وبعد استخدام Strategy Pattern**

#### **قبل التحسين:**

-   الكود مليء بالشروط، مما يجعله صعب الصيانة.
-   عند إضافة نوع جديد، تحتاج لتعديل الكود الرئيسي.
-   الكود غير مرن وغير قابل للتوسعة بسهولة.

#### **بعد التحسين:**

-   تم عزل كل استراتيجية في كود مستقل.
-   إضافة استراتيجية جديدة أصبح بسيطًا دون التأثير على الكود الحالي.
-   الكود أصبح أكثر مرونة وصيانة.

----------

### **الفرق بين Strategy Pattern و Factory Pattern**

----------

#### **1️⃣ التعريف الأساسي لكل نمط:**

-   **Strategy Pattern**:
    
    -   يُستخدم لاختيار وتنفيذ **سلوك أو خوارزمية** معينة بشكل ديناميكي أثناء وقت التشغيل.
    -   يركز على **كيفية التصرف** (Behavior).
-   **Factory Pattern**:
    
    -   يُستخدم لإنشاء كائنات من أنواع مختلفة دون الحاجة لمعرفة النوع الدقيق للكائن في الكود الرئيسي.
    -   يركز على **كيفية إنشاء الكائنات** (Creation).

----------

#### **2️⃣ السؤال الأساسي الذي يجيب عليه كل نمط:**

-   **Strategy Pattern**:
    
    > "كيف يمكنني اختيار وتنفيذ خوارزمية مختلفة بناءً على السيناريو؟"
    
-   **Factory Pattern**:
    
    > "كيف يمكنني إنشاء كائن من نوع محدد دون القلق بشأن تفاصيله الداخلية؟"
    

----------

#### **3️⃣ الفرق في الغرض:**

النمط

الغرض الرئيسي

**Strategy**

تغيير الخوارزميات بسهولة أثناء التشغيل دون تغيير الكود الأساسي.

**Factory**

إنشاء كائنات من أنواع مختلفة بطريقة موحدة ومبسطة.

----------

#### **4️⃣ الفرق في التركيز:**

النمط

يركز على…

**Strategy**

**السلوكيات (Behaviors)**.

**Factory**

**إنشاء الكائنات (Object Creation)**.

----------

#### **5️⃣ كيف يعمل كل نمط؟**

-   **Strategy Pattern**:
    
    1.  يتم تعريف واجهة (Interface) أو كلاس مجرد (Abstract Class) تمثل السلوك أو الخوارزمية.
    2.  يتم تنفيذ السلوكيات المختلفة ككلاسات منفصلة.
    3.  يمكن تغيير السلوك أو الخوارزمية أثناء وقت التشغيل عن طريق اختيار الاستراتيجية المطلوبة.
-   **Factory Pattern**:
    
    1.  يتم تعريف واجهة أو كلاس مجرد لإنشاء الكائنات.
    2.  يتم تنفيذ الكلاسات المختلفة لإنشاء أنواع مختلفة من الكائنات.
    3.  يتم استخدام المصنع (Factory) لاختيار النوع المناسب وإنشاء الكائن.

----------

#### **6️⃣ مثال عملي لكل نمط**

##### **مثال: Strategy Pattern**

**سيناريو:** حساب تكلفة الرحلة بناءً على نوعها.

-   الكلاسات (Strategies):
    
    -   `StandardRideCostStrategy`
    -   `LuxuryRideCostStrategy`
    -   `SharedRideCostStrategy`
-   الكود يختار الاستراتيجية المناسبة بناءً على نوع الرحلة:
    

```csharp
IRideCostStrategy strategy = rideType switch
{
    "Standard" => new StandardRideCostStrategy(),
    "Luxury" => new LuxuryRideCostStrategy(),
    "Shared" => new SharedRideCostStrategy(),
    _ => throw new ArgumentException("Invalid ride type")
};

```

##### **مثال: Factory Pattern**

**سيناريو:** إنشاء كائن يمثل نوع الرحلة.

-   الكلاسات (Products):
    
    -   `StandardRide`
    -   `LuxuryRide`
    -   `SharedRide`
-   المصنع (Factory) يحدد النوع المناسب:
    

```csharp
public IRide CreateRide(string rideType)
{
    return rideType switch
    {
        "Standard" => new StandardRide(),
        "Luxury" => new LuxuryRide(),
        "Shared" => new SharedRide(),
        _ => throw new ArgumentException("Invalid ride type")
    };
}

```

----------

#### **7️⃣ متى نستخدم كل نمط؟**

-   **استخدام Strategy Pattern:**
    
    -   إذا كنت تحتاج إلى تطبيق سلوكيات أو خوارزميات مختلفة في وقت التشغيل.
    -   مثلاً: اختيار طريقة حساب التكلفة، طريقة الدفع، أو أي منطق يتغير بناءً على المدخلات.
-   **استخدام Factory Pattern:**
    
    -   إذا كنت تحتاج إلى إنشاء كائنات متعددة من أنواع مختلفة بدون القلق بشأن كيفية إنشائها.
    -   مثلاً: إنشاء كائن يمثل رحلة بناءً على نوعها، أو إنشاء كائن يمثل نوع مستخدم (Admin, Driver, Customer).

----------

#### **8️⃣ نقاط التشابه:**

-   كلا النمطين يستخدم واجهات (Interfaces) أو كلاسات مجردة (Abstract Classes) لتوحيد العمل.
-   كلاهما يعزز مبدأ **Open/Closed Principle**:
    -   الكود مفتوح للتوسعة بإضافة أنواع جديدة أو استراتيجيات جديدة.
    -   الكود مغلق للتعديل في الأجزاء الحالية.

----------

#### **9️⃣ الفرق الأساسي في Ride-Share Application**

-   **Strategy Pattern:**
    
    -   يتم استخدامه لاختيار الخوارزمية المناسبة (مثل طريقة حساب التكلفة).
    -   يركز على **تنفيذ سلوك مختلف**.
-   **Factory Pattern:**
    
    -   يتم استخدامه لإنشاء الكائنات (مثل إنشاء كائن يمثل نوع الرحلة).
    -   يركز على **إنشاء الكائن المناسب**.

----------

💡 **الخلاصة:**

-   إذا كنت تتعامل مع **سلوكيات وخوارزميات**: استخدم **Strategy Pattern**.
-   إذا كنت تتعامل مع **إنشاء الكائنات**: استخدم **Factory Pattern**.
