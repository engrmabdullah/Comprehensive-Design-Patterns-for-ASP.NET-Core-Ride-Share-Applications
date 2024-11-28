### Composite Pattern 

----------

#### **1️⃣ What is Composite Pattern?**

The **Composite Pattern** allows you to treat individual objects and compositions of objects uniformly. It is especially useful when you have a tree-like structure where:

-   A single object can represent a base entity.
-   A group of objects can represent a composite entity.

----------

#### **Real-World Example:**

Imagine you are managing deliveries.

-   A single box can hold items.
-   A large box can hold smaller boxes, each containing items.  
    To calculate the total weight, you don’t want to distinguish between individual boxes and grouped boxes.

The **Composite Pattern** lets you handle both cases the same way.

----------

#### **2️⃣ Problem Before Using Composite Pattern**

In a Ride-Share application, suppose we want to calculate the total distance of trips:

1.  Some trips are individual (`SingleTrip`).
2.  Other trips are grouped (`GroupTrip`) and include multiple sub-trips.

**Problems:**

1.  The code will be cluttered with conditions to differentiate between individual and group trips.
2.  Adding new types of trips will require modifying multiple parts of the code.

----------

#### 🚫 **Code Without Composite Pattern:**

```csharp
public class SingleTrip
{
    public double Distance { get; set; }

    public double GetTotalDistance()
    {
        return Distance;
    }
}

public class GroupTrip
{
    public List<SingleTrip> Trips { get; set; } = new List<SingleTrip>();

    public double GetTotalDistance()
    {
        double totalDistance = 0;
        foreach (var trip in Trips)
        {
            totalDistance += trip.GetTotalDistance();
        }
        return totalDistance;
    }
}

```

----------

### **❌ Project Structure Before Composite Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   ├── SingleTrip.cs
│   │   └── GroupTrip.cs
└── WebApi/
    └── Controllers/
        └── TripController.cs

```

----------

### **3️⃣ Solution Using Composite Pattern**

----------

#### **The Idea:**

Instead of treating individual and group trips differently:

-   Define a common interface for trips.
-   Both individual trips (`SingleTrip`) and group trips (`GroupTrip`) will implement the same interface.
-   The application can work with any trip type in a unified way.

----------

### **✅ Project Structure After Composite Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── ITripComponent.cs
│   ├── Services/
│   │   ├── SingleTrip.cs
│   │   ├── GroupTrip.cs
└── WebApi/
    └── Controllers/
        └── TripController.cs

```

----------

### **4️⃣ Steps to Implement Composite Pattern**

----------

#### **Step 1: Define a Common Interface**

In the `Application/Interfaces` folder:

```csharp
public interface ITripComponent
{
    double GetTotalDistance();
}

```

**Explanation:**

-   The `ITripComponent` interface defines a method `GetTotalDistance`.
-   Both individual and group trips will implement this interface.

----------

#### **Step 2: Create the Individual Trip Class**

In the `Application/Services` folder:

```csharp
public class SingleTrip : ITripComponent
{
    public double Distance { get; set; }

    public double GetTotalDistance()
    {
        return Distance;
    }
}

```

**Explanation:**

-   `SingleTrip` represents an individual trip and implements `ITripComponent`.

----------

#### **Step 3: Create the Group Trip Class**

```csharp
public class GroupTrip : ITripComponent
{
    private readonly List<ITripComponent> _trips = new List<ITripComponent>();

    public void AddTrip(ITripComponent trip)
    {
        _trips.Add(trip);
    }

    public double GetTotalDistance()
    {
        double totalDistance = 0;
        foreach (var trip in _trips)
        {
            totalDistance += trip.GetTotalDistance();
        }
        return totalDistance;
    }
}

```

**Explanation:**

-   `GroupTrip` represents a collection of trips.
-   It maintains a list of `ITripComponent` objects, allowing it to hold both `SingleTrip` and other `GroupTrip` instances.

----------

#### **Step 4: Create the Service**

```csharp
public class TripService
{
    public double CalculateTotalDistance(ITripComponent trip)
    {
        return trip.GetTotalDistance();
    }
}

```

**Explanation:**

-   `TripService` calculates the total distance of any trip, whether individual or grouped.

----------

#### **Step 5: Create the Controller**

In the `WebApi/Controllers` folder:

```csharp
[ApiController]
[Route("api/[controller]")]
public class TripController : ControllerBase
{
    private readonly TripService _tripService;

    public TripController(TripService tripService)
    {
        _tripService = tripService;
    }

    [HttpGet("total-distance")]
    public IActionResult GetTotalDistance()
    {
        var singleTrip = new SingleTrip { Distance = 10 };
        var anotherSingleTrip = new SingleTrip { Distance = 20 };

        var groupTrip = new GroupTrip();
        groupTrip.AddTrip(singleTrip);
        groupTrip.AddTrip(anotherSingleTrip);

        var totalDistance = _tripService.CalculateTotalDistance(groupTrip);
        return Ok($"Total distance: {totalDistance} km");
    }
}

```

----------

#### **Step 6: Register Services in `Program.cs`**

```csharp
builder.Services.AddScoped<TripService>();

```

----------

### **5️⃣ Benefits of Using Composite Pattern**

#### **Before:**

-   Separate handling for individual and group trips.
-   Adding new trip types required major code changes.

#### **After:**

-   A unified approach for all trip types using `ITripComponent`.
-   Code is more flexible and adheres to the **Open/Closed Principle**:
    -   Open for extension (new trip types can be added).
    -   Closed for modification (existing code does not need changes).

----------


