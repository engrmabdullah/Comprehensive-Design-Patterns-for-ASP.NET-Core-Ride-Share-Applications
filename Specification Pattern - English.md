### Specification Pattern

----------

#### **1️⃣ What is the Specification Pattern?**

The **Specification Pattern** helps define and combine business rules into reusable, testable components. Instead of cluttering the main logic with multiple conditions, the pattern extracts these conditions into separate, reusable objects called specifications.

----------

#### **Real-World Example:**

Imagine you are filtering drivers in a Ride-Share app:

-   Drivers available within a specific radius.
-   Drivers with a high rating.
-   Drivers with available vehicles.

Instead of writing these conditions inline repeatedly, the **Specification Pattern** allows you to define each condition independently and combine them as needed.

----------

#### **2️⃣ Problem Before Using Specification Pattern**

In a Ride-Share app, we need to filter drivers based on:

1.  Availability.
2.  Distance from the user.
3.  Minimum rating.

**Problems:**

1.  The main logic becomes cluttered with multiple `if` or `where` clauses.
2.  Adding new conditions requires modifying existing logic.
3.  Reusing conditions is difficult.

----------

#### 🚫 **Code Without Specification Pattern:**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> GetAvailableDrivers(double userLat, double userLong, double radius, double minRating)
    {
        return _drivers.Where(d =>
            d.IsAvailable &&
            d.Rating >= minRating &&
            CalculateDistance(userLat, userLong, d.Lat, d.Long) <= radius
        ).ToList();
    }

    private double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        var R = 6371; // Earth's radius in kilometers
        var dLat = (lat2 - lat1) * (Math.PI / 180);
        var dLon = (lon2 - lon1) * (Math.PI / 180);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(lat1 * (Math.PI / 180)) * Math.Cos(lat2 * (Math.PI / 180)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
}

```

**Problems:**

1.  The logic is tightly coupled with multiple conditions in one place.
2.  Difficult to add new filters or reuse existing ones.
3.  Not flexible for extensions.

----------

### **❌ Project Structure Before Specification Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── DriverService.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **3️⃣ The Solution: Using Specification Pattern**

----------

#### **The Idea:**

-   Each condition (e.g., "Is Available," "In Radius") is encapsulated into a separate specification object.
-   Specifications can be combined (e.g., `AND`, `OR`) to form complex queries.
-   The main service becomes simpler and reusable.

----------

### **✅ Project Structure After Specification Pattern**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── ISpecification.cs
│   ├── Specifications/
│   │   ├── BaseSpecification.cs
│   │   ├── AvailableDriverSpecification.cs
│   │   ├── DriverInRadiusSpecification.cs
│   │   └── HighRatingDriverSpecification.cs
│   ├── Services/
│   │   └── DriverService.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **4️⃣ Steps to Implement Specification Pattern**

----------

#### **Step 1: Define a Common Specification Interface**

In the `Application/Interfaces` folder:

```csharp
public interface ISpecification<T>
{
    bool IsSatisfiedBy(T entity);
}

```

**Explanation:**

-   The `ISpecification` interface defines a single method, `IsSatisfiedBy`, which determines if an entity meets a specific condition.

----------

#### **Step 2: Create a Base Specification**

In the `Application/Specifications` folder:

```csharp
public abstract class BaseSpecification<T> : ISpecification<T>
{
    public abstract bool IsSatisfiedBy(T entity);

    public BaseSpecification<T> And(BaseSpecification<T> other)
    {
        return new AndSpecification<T>(this, other);
    }
}

```

**Explanation:**

-   This serves as a base class for all specifications, allowing composition using `AND`.

----------

#### **Step 3: Implement Specific Specifications**

##### **Available Driver Specification:**

```csharp
public class AvailableDriverSpecification : BaseSpecification<Driver>
{
    public override bool IsSatisfiedBy(Driver driver)
    {
        return driver.IsAvailable;
    }
}

```

##### **Driver In Radius Specification:**

```csharp
public class DriverInRadiusSpecification : BaseSpecification<Driver>
{
    private readonly double _userLat;
    private readonly double _userLong;
    private readonly double _radius;

    public DriverInRadiusSpecification(double userLat, double userLong, double radius)
    {
        _userLat = userLat;
        _userLong = userLong;
        _radius = radius;
    }

    public override bool IsSatisfiedBy(Driver driver)
    {
        var distance = CalculateDistance(_userLat, _userLong, driver.Lat, driver.Long);
        return distance <= _radius;
    }

    private double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        var R = 6371; // Earth's radius
        var dLat = (lat2 - lat1) * (Math.PI / 180);
        var dLon = (lon2 - lon1) * (Math.PI / 180);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(lat1 * (Math.PI / 180)) * Math.Cos(lat2 * (Math.PI / 180)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
}

```

##### **High Rating Driver Specification:**

```csharp
public class HighRatingDriverSpecification : BaseSpecification<Driver>
{
    private readonly double _minRating;

    public HighRatingDriverSpecification(double minRating)
    {
        _minRating = minRating;
    }

    public override bool IsSatisfiedBy(Driver driver)
    {
        return driver.Rating >= _minRating;
    }
}

```

----------

#### **Step 4: Use Specifications in DriverService**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> GetAvailableDrivers(ISpecification<Driver> specification)
    {
        return _drivers.Where(specification.IsSatisfiedBy).ToList();
    }
}

```

**Explanation:**

-   The `DriverService` applies specifications to filter drivers.

----------

#### **Step 5: Create a Controller**

In the `WebApi/Controllers` folder:

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly DriverService _driverService;

    public DriverController(DriverService driverService)
    {
        _driverService = driverService;
    }

    [HttpGet("available")]
    public IActionResult GetAvailableDrivers(double userLat, double userLong, double radius, double minRating)
    {
        var availableSpec = new AvailableDriverSpecification();
        var radiusSpec = new DriverInRadiusSpecification(userLat, userLong, radius);
        var ratingSpec = new HighRatingDriverSpecification(minRating);

        var combinedSpec = availableSpec.And(radiusSpec).And(ratingSpec);

        var drivers = _driverService.GetAvailableDrivers(combinedSpec);
        return Ok(drivers);
    }
}

```

----------

#### **Step 6: Register Services in `Program.cs`**

```csharp
builder.Services.AddScoped<DriverService>();

```

----------

### **5️⃣ Benefits of Specification Pattern**

#### **Before:**

-   Logic was cluttered with multiple conditions.
-   Difficult to add or reuse filters.

#### **After:**

-   Filters are reusable and modular.
-   Adding new filters is simple and doesn't require modifying existing code.
-   Logic is clean and easy to maintain.

----------


