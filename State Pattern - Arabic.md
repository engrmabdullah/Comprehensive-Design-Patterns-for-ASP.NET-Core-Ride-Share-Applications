### State Pattern

----------

### **1️⃣ يعني إيه State Pattern؟**

يا صديقي، **State Pattern** بيخليك تعبر عن الحالة المختلفة لكائن معين (زي الرحلة في تطبيق مشاركة الركوب) عن طريق فصل كل حالة ككائن مستقل.  
يعني بدل ما نستخدم شرطيات كتير (if-else أو switch-case) علشان نحدد سلوك الكائن بناءً على حالته، بنفصل كل حالة في كائن لوحدها وندير الحالة بسلاسة.

----------

### **2️⃣ المشكلة قبل استخدام State Pattern**

في تطبيق Ride-Share، عندنا حالات كتيرة للرحلة، زي:

-   طلب الرحلة (Requested)
-   قبول الرحلة (Accepted)
-   بدء الرحلة (Started)
-   إنهاء الرحلة (Completed)

الكود التقليدي هيكون مليان شرطيات للتحقق من الحالة الحالية، زي كده:

#### **الكود قبل النمط:**

```csharp
public class Ride
{
    public string Status { get; set; }

    public void Process()
    {
        if (Status == "Requested")
        {
            Console.WriteLine("Processing ride request...");
            Status = "Accepted";
        }
        else if (Status == "Accepted")
        {
            Console.WriteLine("Starting the ride...");
            Status = "Started";
        }
        else if (Status == "Started")
        {
            Console.WriteLine("Completing the ride...");
            Status = "Completed";
        }
        else
        {
            Console.WriteLine("Invalid state!");
        }
    }
}

```

----------

### **المشاكل هنا:**

1.  **شرطيات كتير (If-Else):**  
    الكود هيبقى صعب القراءة والصيانة كل ما زادت الحالات.
    
2.  **عدم تطبيق مبدأ المسؤولية الواحدة (SRP):**  
    كود الـ `Ride` بيحتوي على منطق الحالات المختلفة، وده يخالف مبدأ الفصل بين المسؤوليات.
    
3.  **صعوبة التوسّع:**  
    لو عاوز تضيف حالة جديدة، هتضطر تعدل في الكود وتضيف شرطيات جديدة.
    

----------

### **3️⃣ الحل باستخدام State Pattern**

#### **الفكرة:**

-   كل حالة تبقى كائن مستقل.
-   الرحلة (Ride) تحتفظ بـ "الحالة الحالية" فقط.
-   لما تتغير الحالة، الكائن الحالي يتغير لسلوك جديد.

----------

### **4️⃣ تقسيم المشروع (Root Folder Structure)**

#### **قبل النمط:**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

#### **بعد النمط:**

```plaintext
RootFolder/
├── Application/
│   ├── States/
│   │   ├── RideRequestedState.cs
│   │   ├── RideAcceptedState.cs
│   │   ├── RideStartedState.cs
│   │   └── RideCompletedState.cs
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **5️⃣ الكود بعد تطبيق State Pattern**

----------

#### **الخطوة 1: إنشاء واجهة لكل حالة**

```csharp
public interface IRideState
{
    void Process(Ride ride);
}

```

----------

#### **الخطوة 2: إنشاء الحالات المختلفة**

##### **حالة "Requested":**

```csharp
public class RideRequestedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Processing ride request...");
        ride.SetState(new RideAcceptedState());
    }
}

```

##### **حالة "Accepted":**

```csharp
public class RideAcceptedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Starting the ride...");
        ride.SetState(new RideStartedState());
    }
}

```

##### **حالة "Started":**

```csharp
public class RideStartedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Completing the ride...");
        ride.SetState(new RideCompletedState());
    }
}

```

##### **حالة "Completed":**

```csharp
public class RideCompletedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Ride is already completed.");
    }
}

```

----------

#### **الخطوة 3: إنشاء الكائن Ride**

```csharp
public class Ride
{
    private IRideState _currentState;

    public Ride()
    {
        _currentState = new RideRequestedState(); // الحالة الابتدائية
    }

    public void SetState(IRideState state)
    {
        _currentState = state;
    }

    public void Process()
    {
        _currentState.Process(this);
    }
}

```

----------

#### **الخطوة 4: استخدام النمط**

##### **داخل الـ Controller:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    [HttpPost("process")]
    public IActionResult ProcessRide()
    {
        var ride = new Ride();
        ride.Process(); // Requested -> Accepted
        ride.Process(); // Accepted -> Started
        ride.Process(); // Started -> Completed
        ride.Process(); // Already Completed

        return Ok("Ride processing completed.");
    }
}

```

----------

### **6️⃣ التأثير بعد استخدام State Pattern**

#### **قبل النمط:**

-   الكود معقد وصعب الصيانة بسبب الشرطيات.
-   منطق الحالات موجود داخل الكائن الأساسي (Ride).

#### **بعد النمط:**

-   كل حالة معزولة في كائن خاص بها، مما يسهل الصيانة والتوسيع.
-   كود الـ `Ride` أصبح بسيط وواضح.
-   تم تطبيق مبدأ المسؤولية الواحدة (SRP).

----------

### **7️⃣ استخدامات State Pattern**

1.  **أنظمة تتبع الطلبات:**
    
    -   مثل تطبيقات الطلبات (Requested, In Progress, Delivered).
2.  **أنظمة إدارة المهام:**
    
    -   المهام اللي بتتنقل بين الحالات (To Do, In Progress, Completed).
3.  **أنظمة مشاركة الركوب (Ride-Share):**
    
    -   حالات الرحلة (Requested, Accepted, Started, Completed).
4.  **أنظمة الحجز:**
    
    -   حالات الحجز (Reserved, Checked-In, Checked-Out).

----------


