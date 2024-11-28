### Dependency Injection 

----------

#### **1️⃣ What is Dependency Injection?**

**Dependency Injection (DI)** is a design pattern that separates the logic of a service from its dependencies by "injecting" those dependencies into the service rather than creating them internally.

-   The goal is to make the code more modular, testable, and maintainable.

----------

#### **2️⃣ The Problem Before Using Dependency Injection**

In applications like **Ride-Share**, services often depend on complex dependencies, such as:

1.  Accessing the database.
2.  Interacting with external APIs.
3.  Using other services like cost calculation.

If these dependencies are created directly within the service:

1.  **Difficult Maintenance:** Any change to the dependency requires changes in multiple places.
2.  **Hard Testing:** Testing becomes challenging due to reliance on real dependencies like databases.
3.  **Violation of SRP (Single Responsibility Principle):** Services handle not only their own logic but also manage dependency creation.

----------

#### 🚫 **Code Without Dependency Injection (Code Without Pattern):**

```csharp
public class RideCostService
{
    private readonly RideDatabase _rideDatabase;

    public RideCostService()
    {
        _rideDatabase = new RideDatabase(); // Dependency is created internally
    }

    public double CalculateRideCost(int rideId)
    {
        var ride = _rideDatabase.GetRideById(rideId);
        return ride.Distance * 1.5;
    }
}

```

**Problems with this code:**

1.  Cannot replace `RideDatabase` during testing.
2.  Any change in how data is fetched requires changes in the service itself.
3.  Code is tightly coupled and inflexible.

----------

### **❌ Project Structure Before Applying Dependency Injection:**

```plaintext
RootFolder/
├── Services/
│   └── RideCostService.cs
├── Data/
│   └── RideDatabase.cs
└── Program.cs

```

----------

### **3️⃣ The Solution: Using Dependency Injection**

With **Dependency Injection**, dependencies are passed into the service from the outside rather than being created internally.  
This results in a clear separation of concerns and makes the code more flexible and testable.

----------

### **✅ Project Structure After Applying Dependency Injection:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideDatabase.cs
│   ├── Services/
│   │   └── RideCostService.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Data/
│   │   └── RideDatabase.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement Dependency Injection**

----------

#### **Step 1: Define an Interface for the Dependency**

In `Application/Interfaces`:

```csharp
public interface IRideDatabase
{
    Ride GetRideById(int rideId);
}

```

----------

#### **Step 2: Implement the Dependency in a Separate Class**

In `Infrastructure/Data`:

```csharp
public class RideDatabase : IRideDatabase
{
    public Ride GetRideById(int rideId)
    {
        // Simulate fetching ride data from the database
        return new Ride { Id = rideId, Distance = 10.0 };
    }
}

```

----------

#### **Step 3: Inject the Dependency in the Service**

In `Application/Services`:

```csharp
public class RideCostService
{
    private readonly IRideDatabase _rideDatabase;

    // Dependency is injected here
    public RideCostService(IRideDatabase rideDatabase)
    {
        _rideDatabase = rideDatabase;
    }

    public double CalculateRideCost(int rideId)
    {
        var ride = _rideDatabase.GetRideById(rideId);
        return ride.Distance * 1.5;
    }
}

```

----------

#### **Step 4: Define the Ride Entity**

In `Domain/Entities`:

```csharp
public class Ride
{
    public int Id { get; set; }
    public double Distance { get; set; }
}

```

----------

#### **Step 5: Create a Controller to Use the Service**

In `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly RideCostService _rideCostService;

    public RideController(RideCostService rideCostService)
    {
        _rideCostService = rideCostService;
    }

    [HttpGet("cost/{rideId}")]
    public IActionResult GetRideCost(int rideId)
    {
        var cost = _rideCostService.CalculateRideCost(rideId);
        return Ok(cost);
    }
}

```

----------

#### **Step 6: Register the Dependencies in `Program.cs`**

```csharp
builder.Services.AddScoped<IRideDatabase, RideDatabase>();
builder.Services.AddScoped<RideCostService>();

```

----------

### **5️⃣ How Did We Overcome the Challenges?**

----------

#### **1. Difficulty in Testing Due to Real Dependencies**

By using an interface (`IRideDatabase`), the real dependency (`RideDatabase`) can be replaced with a **Mock** or **Stub** during testing.

##### **Example of Testing with a Mock:**

```csharp
var mockRideDatabase = new Mock<IRideDatabase>();
mockRideDatabase.Setup(db => db.GetRideById(It.IsAny<int>()))
                .Returns(new Ride { Id = 1, Distance = 20.0 });

var service = new RideCostService(mockRideDatabase.Object);

var cost = service.CalculateRideCost(1);

Assert.Equal(30.0, cost); // 20 * 1.5

```

----------

#### **2. Difficulty in Maintenance When Dependencies Change**

-   If the way data is retrieved changes (e.g., from a database to an API), we only need to implement the new logic in another class without modifying the service.

##### **Adding a New Data Source:**

```csharp
public class RideApiDatabase : IRideDatabase
{
    public Ride GetRideById(int rideId)
    {
        // Fetch ride data from an API
        return new Ride { Id = rideId, Distance = 15.0 };
    }
}

```

**Update Registration in `Program.cs`:**

```csharp
builder.Services.AddScoped<IRideDatabase, RideApiDatabase>();

```

----------

#### **3. Improved Code Organization and Cleanliness**

-   The service (`RideCostService`) now only focuses on its core logic and delegates data fetching to the dependency.
-   This adheres to the **Single Responsibility Principle (SRP)** and makes the code easier to read and maintain.

----------

### **6️⃣ Impact of Using Dependency Injection**

----------

#### **Before:**

-   Dependencies were tightly coupled to services.
-   Testing required setting up real databases or external services.
-   Code changes in dependencies required modifying multiple classes.

#### **After:**

-   Dependencies are injected, making them easy to replace and test.
-   Mock objects can be used for faster and isolated tests.
-   Code follows SOLID principles, making it modular and maintainable.

----------

