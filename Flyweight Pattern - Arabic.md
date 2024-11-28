###  Flyweight Pattern

----------

#### **إيه هو Flyweight Pattern؟**

يا صديقي، Flyweight Pattern ده بنستخدمه لما يكون عندك كائنات كتير جدًا متكررة، وعايز تقلل استخدام الذاكرة عن طريق إعادة استخدام الكائنات بدل ما تعمل كائن جديد لكل واحدة. ببساطة هو بيساعدنا نوفر في **الذاكرة**.

----------

#### **تخيل السيناريو ده**

عندك تطبيق لمشاركة الركوب (Ride Share)، وبتعرض **خريطة** للسائقين حواليك. كل سائق ليه لون مختلف على الخريطة. دلوقتي لو عندك 10,000 سائق، هتضطر تعمل 10,000 كائن علشان تخزن البيانات دي. وده هيستهلك **ذاكرة ضخمة**!

----------

#### **المشكلة قبل استخدام Flyweight Pattern**

لو كتبت الكود بشكل مباشر:

```csharp
public class DriverMarker
{
    public string Color { get; set; }
    public string Position { get; set; }

    public DriverMarker(string color, string position)
    {
        Color = color;
        Position = position;
    }
}

```

لما تعمل كائن لكل سائق:

```csharp
var markers = new List<DriverMarker>();
for (int i = 0; i < 10000; i++)
{
    markers.Add(new DriverMarker("Red", $"Position {i}"));
}

```

**المشكلة:**

-   **ذاكرة كتير جدًا مستخدمة.**
-   **تكرار في الكائنات.**

----------

#### **الحل باستخدام Flyweight Pattern**

هنعيد استخدام الكائنات اللي ليها نفس الخصائص المتكررة (زي اللون).

----------

### **1. تقسيم المشروع (Root Folder Structure)**

```plaintext
RideShareApp/
├── Domain/
│   ├── Entities/
│   │   └── DriverMarker.cs
│   └── Interfaces/
│       └── IDriverMarkerFactory.cs
├── Application/
│   ├── Flyweight/
│   │   ├── DriverMarkerFactory.cs
│   │   └── DriverMarkerFlyweight.cs
└── Program.cs

```

----------

### **2. الكود باستخدام Flyweight Pattern**

#### **الخطوة 1: تعريف كائن Marker (Flyweight Object)**

```csharp
public class DriverMarker
{
    public string Color { get; private set; }

    public DriverMarker(string color)
    {
        Color = color;
    }

    public void Display(string position)
    {
        Console.WriteLine($"Driver at {position} with color {Color}");
    }
}

```

----------

#### **الخطوة 2: إنشاء Factory لإدارة الكائنات المتكررة**

```csharp
public class DriverMarkerFactory
{
    private Dictionary<string, DriverMarker> _markers = new Dictionary<string, DriverMarker>();

    public DriverMarker GetMarker(string color)
    {
        if (!_markers.ContainsKey(color))
        {
            _markers[color] = new DriverMarker(color);
        }
        return _markers[color];
    }
}

```

----------

#### **الخطوة 3: استخدام Flyweight في الكود الرئيسي**

```csharp
class Program
{
    static void Main(string[] args)
    {
        var factory = new DriverMarkerFactory();

        for (int i = 0; i < 100; i++)
        {
            var marker = factory.GetMarker("Red");
            marker.Display($"Position {i}");
        }

        for (int i = 0; i < 100; i++)
        {
            var marker = factory.GetMarker("Blue");
            marker.Display($"Position {i}");
        }
    }
}

```

----------

### **3. النتائج عند تشغيل الكود**

```plaintext
Driver at Position 0 with color Red
Driver at Position 1 with color Red
Driver at Position 2 with color Red
...
Driver at Position 0 with color Blue
Driver at Position 1 with color Blue
Driver at Position 2 with color Blue
...

```

----------

### **4. مميزات Flyweight Pattern**

1.  **توفير الذاكرة**: الكائنات اللي ليها نفس الخصائص المتكررة بتتخزن مرة واحدة بس.
2.  **سهولة الإدارة**: إدارة الكائنات المتكررة بقت أسهل باستخدام Factory.
3.  **تحسين الأداء**: استخدام أقل للذاكرة بيساهم في تحسين الأداء.

----------

### **5. أمثلة استخدام Flyweight Pattern**

1.  **تطبيقات الخرائط**:
    -   إعادة استخدام رموز الأماكن (Markers) بدل ما نعمل كائن جديد لكل رمز.
2.  **ألعاب الفيديو**:
    -   الكائنات المتكررة زي الأشجار أو المنازل.
3.  **تحليل النصوص**:
    -   تخزين الأحرف المتكررة في النصوص الطويلة كـ Flyweights.

----------


