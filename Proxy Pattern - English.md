### Proxy Pattern

----------

#### **1️⃣ What is Proxy Pattern?**

The **Proxy Pattern** introduces a mediator or substitute object, called a "proxy," to control access to a target object. The proxy can perform various functions, such as:

-   Adding caching for improved performance.
-   Controlling access to heavy or sensitive resources.
-   Enhancing functionality without modifying the original object.

----------

#### **Real-World Example:**

Imagine you need a book from a library. Instead of searching for it yourself, you approach a librarian (proxy), who fetches the book for you. If the book is frequently requested, the librarian might keep it readily available to save time.

----------

#### **2️⃣ Problem Before Using Proxy Pattern**

In a Ride-Share application, suppose you have a service like `RideCostCalculator` that fetches ride costs directly from a database or external API.

**Problems:**

1.  Each request triggers a database query, even for the same data.
2.  There’s no caching, leading to unnecessary resource consumption and slower performance.

----------

#### 🚫 **Code Without Proxy Pattern:**

```csharp
public class RideCostCalculator
{
    public double GetCost(int rideId)
    {
        // Simulating a database query
        Console.WriteLine($"Fetching cost for ride {rideId} from the database...");
        return 50.0; // Example ride cost
    }
}

```

----------

### **❌ Project Structure Before Proxy Pattern**

```plaintext
RootFolder/
├── Application/
│   └── Services/
│       └── RideCostCalculator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

**Issues:**

1.  Every time `GetCost` is called, it queries the database, even for the same ride ID.
2.  Redundant queries slow down the application.

----------

### **3️⃣ The Solution: Using Proxy Pattern**

----------

#### **The Idea:**

Introduce a **Proxy** to act as an intermediary:

1.  The Proxy checks if the requested data is already cached.
2.  If it is, it returns the cached data.
3.  If not, it fetches the data from the main class (`RideCostCalculator`) and caches it for future use.

----------

### **✅ Project Structure After Proxy Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostCalculator.cs
│   ├── Services/
│   │   ├── RideCostCalculator.cs
│   │   └── RideCostProxy.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ Steps to Implement Proxy Pattern**

----------

#### **Step 1: Create a Common Interface**

In the `Application/Interfaces` folder:

```csharp
public interface IRideCostCalculator
{
    double GetCost(int rideId);
}

```

**Explanation:**

-   The interface defines the contract for both the core class and the proxy.
-   This ensures that the service can use either implementation interchangeably.

----------

#### **Step 2: Create the Core Class**

In the `Application/Services` folder:

```csharp
public class RideCostCalculator : IRideCostCalculator
{
    public double GetCost(int rideId)
    {
        Console.WriteLine($"Fetching cost for ride {rideId} from the database...");
        return 50.0; // Example cost
    }
}

```

**Explanation:**

-   This is the main class that fetches ride costs directly from the database.

----------

#### **Step 3: Create the Proxy**

In the same folder:

```csharp
public class RideCostProxy : IRideCostCalculator
{
    private readonly IRideCostCalculator _calculator;
    private readonly Dictionary<int, double> _cache = new();

    public RideCostProxy(IRideCostCalculator calculator)
    {
        _calculator = calculator;
    }

    public double GetCost(int rideId)
    {
        if (_cache.ContainsKey(rideId))
        {
            Console.WriteLine($"Returning cached cost for ride {rideId}...");
            return _cache[rideId];
        }

        var cost = _calculator.GetCost(rideId);
        _cache[rideId] = cost;
        return cost;
    }
}

```

**Explanation:**

-   The Proxy checks if the ride cost is already cached.
-   If cached, it returns the stored value.
-   If not, it queries the core class, stores the result in the cache, and returns it.

----------

#### **Step 4: Use the Proxy in RideService**

```csharp
public class RideService
{
    private readonly IRideCostCalculator _costCalculator;

    public RideService(IRideCostCalculator costCalculator)
    {
        _costCalculator = costCalculator;
    }

    public double GetRideCost(int rideId)
    {
        return _costCalculator.GetCost(rideId);
    }
}

```

**Explanation:**

-   `RideService` depends on the `IRideCostCalculator` interface, making it flexible to use either the core class or the proxy.

----------

#### **Step 5: Create the Controller**

In the `WebApi/Controllers` folder:

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

    [HttpGet("cost/{rideId}")]
    public IActionResult GetCost(int rideId)
    {
        var cost = _rideService.GetRideCost(rideId);
        return Ok($"The cost of ride {rideId} is {cost} USD");
    }
}

```

----------

#### **Step 6: Register Services in `Program.cs`**

```csharp
builder.Services.AddScoped<RideCostCalculator>();
builder.Services.AddScoped<IRideCostCalculator, RideCostProxy>();
builder.Services.AddScoped<RideService>();

```

**Explanation:**

-   The Proxy is registered as the implementation of `IRideCostCalculator`.
-   This allows the application to use the Proxy wherever the interface is required.

----------

### **5️⃣ Benefits of Using Proxy Pattern**

#### **Before:**

-   Every call to `GetCost` resulted in a new database query.
-   Resource consumption was unnecessarily high.

#### **After:**

-   Frequently requested data is served from the cache, reducing database queries.
-   The application is more efficient and faster.
-   The code adheres to the **Open/Closed Principle**:
    -   Open for extension (adding caching).
    -   Closed for modification (no changes to the original class).

----------

