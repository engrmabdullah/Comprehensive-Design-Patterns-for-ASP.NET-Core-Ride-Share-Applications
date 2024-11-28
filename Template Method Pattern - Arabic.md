###  Template Method Pattern

----------

#### **إيه هو Template Method Pattern؟**

يا صديقي، ده باترن بنستخدمه لما يكون عندنا خطوات متكررة أو مشتركة بين كذا كلاس، بس مش كل الخطوات بتتنفذ بنفس الطريقة. يعني عندك شوية خطوات عامة، بس ممكن كل كلاس يغير في تفاصيل معينة من الخطوات دي.

----------

#### **مثال من حياتنا اليومية**

تخيل إنك عندك عربية، وكل مرة بتحضرها للسفر بتعمل شوية خطوات عامة زي:

1.  **فحص الزيت**.
2.  **تعبئة البنزين**.
3.  **تشغيل الموتور**.

لكن لو العربية دي **عادية** أو **كهربائية**، ممكن طريقة تشغيل الموتور تختلف، بس باقي الخطوات زي ما هي.

----------

#### **المشكلة قبل الباترن**

لو كتبت كل الخطوات لكل نوع عربية في كود مستقل، هيكون عندك تكرار كتير جدًا:

```csharp
public class GasolineCar
{
    public void PrepareForTrip()
    {
        Console.WriteLine("Checking oil...");
        Console.WriteLine("Filling up gasoline...");
        Console.WriteLine("Starting the gasoline engine...");
    }
}

public class ElectricCar
{
    public void PrepareForTrip()
    {
        Console.WriteLine("Checking oil...");
        Console.WriteLine("Charging the battery...");
        Console.WriteLine("Starting the electric engine...");
    }
}

```

----------

#### **ليه ده غلط؟**

1.  **تكرار الكود**: فحص الزيت موجود في كل كلاس.
2.  **صعوبة التعديل**: لو عايز تضيف خطوة جديدة زي "فحص الكفرات"، هتضطر تعدل في كل كلاس.

----------

#### **الحل باستخدام Template Method Pattern**

هنفصل الخطوات المشتركة في كلاس أساسي (**Base Class**)، وهنخلي الخطوات اللي ممكن تتغير عبارة عن دوال قابلة للتنفيذ في الكلاسات الوريثة (**Overridable Methods**).

----------

### **1. تقسيم المشروع (Root Folder Structure)**

```plaintext
RideShareApp/
├── Domain/
│   └── Entities/
│       └── Ride.cs
├── Application/
│   └── RidePreparation/
│       ├── RidePreparationTemplate.cs
│       ├── GasolineRidePreparation.cs
│       └── ElectricRidePreparation.cs
├── Infrastructure/
└── Program.cs

```

----------

### **2. الكود باستخدام Template Method Pattern**

#### **الخطوة 1: تعريف Template Class**

```csharp
public abstract class RidePreparationTemplate
{
    // Template method
    public void PrepareRide()
    {
        CheckOil();
        Refuel();
        StartEngine();
    }

    protected void CheckOil()
    {
        Console.WriteLine("Checking oil...");
    }

    // Abstract methods to be implemented by subclasses
    protected abstract void Refuel();
    protected abstract void StartEngine();
}

```

----------

#### **الخطوة 2: تنفيذ GasolineRidePreparation**

```csharp
public class GasolineRidePreparation : RidePreparationTemplate
{
    protected override void Refuel()
    {
        Console.WriteLine("Filling up gasoline...");
    }

    protected override void StartEngine()
    {
        Console.WriteLine("Starting the gasoline engine...");
    }
}

```

----------

#### **الخطوة 3: تنفيذ ElectricRidePreparation**

```csharp
public class ElectricRidePreparation : RidePreparationTemplate
{
    protected override void Refuel()
    {
        Console.WriteLine("Charging the battery...");
    }

    protected override void StartEngine()
    {
        Console.WriteLine("Starting the electric engine...");
    }
}

```

----------

#### **الخطوة 4: استخدام الكود في Main**

```csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Preparing a gasoline car:");
        RidePreparationTemplate gasolineRide = new GasolineRidePreparation();
        gasolineRide.PrepareRide();

        Console.WriteLine("\nPreparing an electric car:");
        RidePreparationTemplate electricRide = new ElectricRidePreparation();
        electricRide.PrepareRide();
    }
}

```

----------

### **3. النتائج عند تشغيل الكود**

```plaintext
Preparing a gasoline car:
Checking oil...
Filling up gasoline...
Starting the gasoline engine...

Preparing an electric car:
Checking oil...
Charging the battery...
Starting the electric engine...

```

----------

### **4. مميزات Template Method Pattern**

1.  **إعادة استخدام الكود**: الكود المشترك موجود في الكلاس الأساسي.
2.  **مرونة التعديل**: يمكن تغيير خطوات معينة فقط دون التأثير على باقي الكود.
3.  **تقليل التكرار**: كل الخطوات المشتركة موجودة في مكان واحد.

----------

### **5. أمثلة استخدامات Template Method Pattern**

1.  **تحضير الطلبات في التجارة الإلكترونية**:
    
    -   خطوات مشتركة: التحقق من الدفع، تحضير الشحنة.
    -   خطوات متغيرة: طريقة الشحن (سريع أو عادي).
2.  **إعداد المركبات**:
    
    -   خطوات مشتركة: فحص الزيت، فحص العجلات.
    -   خطوات متغيرة: نوع الوقود أو الطاقة.
3.  **معالجة البيانات**:
    
    -   خطوات مشتركة: قراءة البيانات، التحقق من الصيغة.
    -   خطوات متغيرة: نوع التحليل (تحليل نصوص أو صور).

----------


