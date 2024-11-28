### Flyweight Pattern

----------

#### **What is the Flyweight Pattern?**

The **Flyweight Pattern** is a structural design pattern used to reduce memory usage by sharing common data across multiple objects. It’s especially helpful when you have a large number of similar objects that can share their immutable (unchanging) parts.

----------

#### **Why Do We Need Flyweight?**

Imagine a ride-share app where a map displays thousands of drivers as markers. Each marker has a **color** and **position**. Without Flyweight, we’d create a separate object for every driver, consuming a huge amount of memory.

----------

#### **The Problem Without Flyweight**

Let’s consider a basic implementation:

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

If you create a marker for 10,000 drivers:

```csharp
var markers = new List<DriverMarker>();
for (int i = 0; i < 10000; i++)
{
    markers.Add(new DriverMarker("Red", $"Position {i}"));
}

```

**Problems:**

1.  **High Memory Usage**: Each marker duplicates the same color data.
2.  **Inefficiency**: Creating and storing thousands of objects is unnecessary.

----------

#### **The Solution: Flyweight Pattern**

Using Flyweight, we can share the **color** property across multiple markers and only store the unique properties (like position) separately.

----------

### **1. Project Structure**

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

### **2. Implementation**

#### **Step 1: Define the Flyweight Object**

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

#### **Step 2: Create a Factory for Flyweight Objects**

The factory ensures that we reuse existing markers with the same color instead of creating new ones.

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

#### **Step 3: Use Flyweight in the Main Program**

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

### **3. Output**

The output will look like this:

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

### **4. Advantages of Flyweight Pattern**

1.  **Memory Efficiency**:
    
    -   Shared data (e.g., color) is stored only once.
    -   Unique data (e.g., position) is stored externally.
2.  **Performance Improvement**:
    
    -   Reduces object creation overhead.
3.  **Flexibility**:
    
    -   Centralized control over shared data through the factory.

----------

### **5. Real-Life Use Cases**

1.  **Maps**:
    
    -   Markers (e.g., driver locations) with shared attributes like color.
2.  **Games**:
    
    -   Objects like trees or buildings with shared textures.
3.  **Text Editors**:
    
    -   Flyweight is used to store repeated characters.

----------

