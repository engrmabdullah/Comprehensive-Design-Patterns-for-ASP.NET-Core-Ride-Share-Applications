###  Builder Pattern 

----------

#### **1️⃣ What is the Builder Pattern?**

Imagine ordering a car from a manufacturer. Instead of knowing the details of how the car is assembled, you simply specify, "I want a car with four seats, a panoramic roof, and a 2000cc engine." The manufacturer has an engineer who builds the car step-by-step and delivers the final product.

Similarly, the **Builder Pattern**:

-   Separates the construction of a complex object from its representation.
-   Simplifies the creation of objects with multiple configurations by using step-by-step methods.

----------

#### **2️⃣ Problem Before Using Builder Pattern**

In a **Ride-Share** application, when creating an object like `Ride` with many details:

1.  Driver (`DriverId`).
2.  User (`UserId`).
3.  Start and end points (`StartPoint`, `EndPoint`).
4.  Distance (`Distance`).
5.  Price (`Price`).

Creating such objects in one go can make the code:

-   Hard to read.
-   Difficult to maintain.

----------

#### 🚫 **Code Without Builder Pattern:**

```csharp
public class Ride
{
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public string StartPoint { get; set; }
    public string EndPoint { get; set; }
    public double Distance { get; set; }
    public double Price { get; set; }

    public Ride(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        DriverId = driverId;
        UserId = userId;
        StartPoint = startPoint;
        EndPoint = endPoint;
        Distance = distance;
        Price = price;
    }
}

```

----------

#### **Usage in RideService:**

```csharp
var ride = new Ride(1, 2, "Point A", "Point B", 15.5, 30.0);

```

----------

**Problems:**

1.  As the number of properties increases, constructors become cumbersome.
2.  Default values for some properties require multiple constructors.
3.  Hard to read and maintain.

----------

### **❌ Project Structure Before Builder Pattern:**

```plaintext
RootFolder/
├── Domain/
│   └── Entities/
│       └── Ride.cs
└── Application/
    └── Services/
        └── RideService.cs

```

----------

### **3️⃣ Solution: Using Builder Pattern**

----------

### **✅ Project Structure After Builder Pattern:**

```plaintext
RootFolder/
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
│   └── Builders/
│       └── RideBuilder.cs
└── Application/
    └── Services/
        └── RideService.cs

```

----------

### **4️⃣ Steps to Implement Builder Pattern**

----------

#### **Step 1: Define the Ride Class**

```csharp
public class Ride
{
    public int DriverId { get; private set; }
    public int UserId { get; private set; }
    public string StartPoint { get; private set; }
    public string EndPoint { get; private set; }
    public double Distance { get; private set; }
    public double Price { get; private set; }

    public Ride(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        DriverId = driverId;
        UserId = userId;
        StartPoint = startPoint;
        EndPoint = endPoint;
        Distance = distance;
        Price = price;
    }
}

```

**Explanation:**

-   The `Ride` class represents the object we want to build.
-   Properties are marked as `private set` to ensure immutability after object creation.

----------

#### **Step 2: Create the Builder for Ride**

In the `Domain/Builders` folder:

```csharp
public class RideBuilder
{
    private int _driverId;
    private int _userId;
    private string _startPoint;
    private string _endPoint;
    private double _distance;
    private double _price;

    public RideBuilder SetDriver(int driverId)
    {
        _driverId = driverId;
        return this;
    }

    public RideBuilder SetUser(int userId)
    {
        _userId = userId;
        return this;
    }

    public RideBuilder SetStartPoint(string startPoint)
    {
        _startPoint = startPoint;
        return this;
    }

    public RideBuilder SetEndPoint(string endPoint)
    {
        _endPoint = endPoint;
        return this;
    }

    public RideBuilder SetDistance(double distance)
    {
        _distance = distance;
        return this;
    }

    public RideBuilder SetPrice(double price)
    {
        _price = price;
        return this;
    }

    public Ride Build()
    {
        return new Ride(_driverId, _userId, _startPoint, _endPoint, _distance, _price);
    }
}

```

**Explanation:**

1.  `RideBuilder` is responsible for constructing the `Ride` object.
2.  Each property has a `Set` method for assigning values.
3.  `Build()` method creates the final `Ride` object.

----------

#### **Step 3: Use the Builder in RideService**

In `Application/Services`:

```csharp
public class RideService
{
    public Ride CreateRide(int driverId, int userId, string startPoint, string endPoint, double distance, double price)
    {
        var rideBuilder = new RideBuilder();

        var ride = rideBuilder
            .SetDriver(driverId)
            .SetUser(userId)
            .SetStartPoint(startPoint)
            .SetEndPoint(endPoint)
            .SetDistance(distance)
            .SetPrice(price)
            .Build();

        return ride;
    }
}

```

**Explanation:**

-   `RideBuilder` is used to set the necessary properties step-by-step.
-   This improves readability and modularity.

----------

#### **Step 4: Create the Controller**

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

    [HttpPost("create")]
    public IActionResult CreateRide([FromBody] CreateRideDto dto)
    {
        var ride = _rideService.CreateRide(dto.DriverId, dto.UserId, dto.StartPoint, dto.EndPoint, dto.Distance, dto.Price);
        return Ok(ride);
    }
}

```

**Explanation:**

-   The controller calls the `RideService` to create a ride using the `RideBuilder`.

----------

#### **Step 5: Register Services in `Program.cs`**

```csharp
builder.Services.AddScoped<RideService>();

```

----------

### **5️⃣ Impact of Using Builder Pattern**

#### **Before:**

-   Creating complex objects required multiple constructors or large initialization code.
-   Difficult to read and maintain.

#### **After:**

-   Objects are constructed step-by-step in a modular way.
-   Code is more readable and easier to maintain.
-   Adding new properties or configurations is straightforward without affecting existing code.

----------


