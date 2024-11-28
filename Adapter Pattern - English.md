### Adapter Pattern 

----------

#### **1️⃣ What is the Adapter Pattern?**

Imagine you buy an electronic device from the US, but the plug doesn't fit your local power outlet. To make it work, you use an adapter that converts the plug to be compatible with your outlet.

Similarly, the **Adapter Pattern**:

-   Acts as a bridge between two incompatible interfaces.
-   Enables different components to work together without changing their existing code.

----------

#### **2️⃣ Problem Before Using Adapter Pattern**

In a **Ride-Share** application, let’s say you want to add a **distance calculation feature** using two different libraries:

1.  One library uses geographic coordinates (`Latitude`, `Longitude`).
2.  Another library uses textual map locations (`StartPoint`, `EndPoint`).

**The Problem:**  
Both libraries have different interfaces, making them incompatible for direct use in your service.

----------

#### 🚫 **Code Without Adapter Pattern:**

```csharp
// Library for distance calculation using Lat/Long
public class LatLongDistanceCalculator
{
    public double Calculate(double lat1, double lon1, double lat2, double lon2)
    {
        // Logic to calculate distance
        return Math.Sqrt(Math.Pow(lat2 - lat1, 2) + Math.Pow(lon2 - lon1, 2));
    }
}

// Library for distance calculation using map points
public class MapDistanceCalculator
{
    public double GetDistance(string startPoint, string endPoint)
    {
        // Logic to calculate distance
        return 15.0; // Mocked value
    }
}

// RideService attempts to use one of the libraries
public class RideService
{
    private readonly LatLongDistanceCalculator _latLongCalculator;

    public RideService(LatLongDistanceCalculator latLongCalculator)
    {
        _latLongCalculator = latLongCalculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _latLongCalculator.Calculate(lat1, lon1, lat2, lon2);
    }
}

```

----------

#### **Problems:**

1.  If you want to switch to another library (e.g., `MapDistanceCalculator`), you must modify the service code.
2.  The code becomes tightly coupled and less flexible.

----------

### **❌ Project Structure Before Adapter Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideService.cs
├── Libraries/
│   ├── LatLongDistanceCalculator.cs
│   └── MapDistanceCalculator.cs
└── Program.cs

```

----------

### **3️⃣ The Solution: Using Adapter Pattern**

----------

#### **The Idea:**

We create a common **interface** for all distance calculators, then implement **adapters** for each library, ensuring compatibility without changing the existing libraries.

----------

### **✅ Project Structure After Adapter Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IDistanceCalculator.cs
│   ├── Adapters/
│   │   ├── LatLongAdapter.cs
│   │   └── MapAdapter.cs
├── Libraries/
│   ├── LatLongDistanceCalculator.cs
│   └── MapDistanceCalculator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ Steps to Implement Adapter Pattern**

----------

#### **Step 1: Define a Common Interface**

In the `Application/Interfaces` folder:

```csharp
public interface IDistanceCalculator
{
    double CalculateDistance(double lat1, double lon1, double lat2, double lon2);
}

```

**Explanation:**

-   `IDistanceCalculator` defines a common method `CalculateDistance`.
-   This ensures a unified way to interact with different distance calculation libraries.

----------

#### **Step 2: Create Adapters for Each Library**

----------

**Adapter for `LatLongDistanceCalculator`:**

```csharp
public class LatLongAdapter : IDistanceCalculator
{
    private readonly LatLongDistanceCalculator _calculator;

    public LatLongAdapter(LatLongDistanceCalculator calculator)
    {
        _calculator = calculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _calculator.Calculate(lat1, lon1, lat2, lon2);
    }
}

```

----------

**Adapter for `MapDistanceCalculator`:**

```csharp
public class MapAdapter : IDistanceCalculator
{
    private readonly MapDistanceCalculator _calculator;

    public MapAdapter(MapDistanceCalculator calculator)
    {
        _calculator = calculator;
    }

    public double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        // Convert coordinates to textual map points
        string startPoint = $"{lat1},{lon1}";
        string endPoint = $"{lat2},{lon2}";
        return _calculator.GetDistance(startPoint, endPoint);
    }
}

```

**Explanation:**

-   Each adapter implements the `IDistanceCalculator` interface, making both libraries compatible.

----------

#### **Step 3: Use Adapters in RideService**

In the `Application/Services` folder:

```csharp
public class RideService
{
    private readonly IDistanceCalculator _distanceCalculator;

    public RideService(IDistanceCalculator distanceCalculator)
    {
        _distanceCalculator = distanceCalculator;
    }

    public double GetRideDistance(double lat1, double lon1, double lat2, double lon2)
    {
        return _distanceCalculator.CalculateDistance(lat1, lon1, lat2, lon2);
    }
}

```

**Explanation:**

-   `RideService` now works with `IDistanceCalculator`, making it independent of any specific library.

----------

#### **Step 4: Register Services in `Program.cs`**

```csharp
builder.Services.AddSingleton<LatLongDistanceCalculator>();
builder.Services.AddSingleton<IDistanceCalculator, LatLongAdapter>();
builder.Services.AddScoped<RideService>();

```

**Explanation:**

-   You can replace `LatLongAdapter` with `MapAdapter` without changing the service or controller.

----------

#### **Step 5: Create the Controller**

In `WebApi/Controllers`:

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

    [HttpGet("distance")]
    public IActionResult GetDistance(double lat1, double lon1, double lat2, double lon2)
    {
        var distance = _rideService.GetRideDistance(lat1, lon1, lat2, lon2);
        return Ok($"The distance is {distance} km");
    }
}

```

**Explanation:**

-   The controller simply calls the `RideService`, which uses the adapter to calculate the distance.

----------

### **5️⃣ Impact of Adapter Pattern**

#### **Before:**

-   `RideService` directly depended on specific libraries.
-   Adding new libraries required modifying the service code.

#### **After:**

-   `RideService` depends on the common `IDistanceCalculator` interface, making it flexible.
-   Adding a new library only requires creating a new adapter, without touching the existing code.
-   Adheres to the **Open/Closed Principle** (open for extension, closed for modification).

----------


