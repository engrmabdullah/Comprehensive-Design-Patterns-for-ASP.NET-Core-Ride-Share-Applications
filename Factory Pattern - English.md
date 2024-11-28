### Factory Pattern 

----------

#### **1️⃣ What is Factory Pattern?**

The **Factory Pattern** is a design pattern used to create objects without specifying their exact class in the code.  
It acts like a workshop where you can get objects created for you, so you don’t have to deal with the details.

----------

### **2️⃣ Problem: Creating Objects Manually is Inefficient**

When creating different types of objects (e.g., ride types like Standard, Luxury, or Shared), you may encounter issues such as:

1.  **Code Duplication:** Repeating the logic for object creation in multiple places.
2.  **Maintenance Challenges:** If the creation logic changes, you need to update it everywhere.
3.  **Lack of Flexibility:** Adding new types becomes complicated and can break existing code.

----------

#### 🛑 **Code Before Using Factory Pattern (Code Without Pattern):**

```csharp
public class RideService
{
    public IRide GetRide(string rideType)
    {
        if (rideType == "Standard")
        {
            return new StandardRide();
        }
        else if (rideType == "Luxury")
        {
            return new LuxuryRide();
        }
        else if (rideType == "Shared")
        {
            return new SharedRide();
        }
        else
        {
            throw new ArgumentException("Invalid ride type!");
        }
    }
}

```

-   **What’s wrong with this code?**  
    Every time you want to add a new ride type, you’ll need to modify this logic, which can lead to errors and repeated code.

----------

### **❌ Project Structure Before Applying Clean Architecture:**

```plaintext
RootFolder/
├── Data/
│   ├── RideShareDbContext.cs
├── Services/
│   └── RideService.cs
├── Entities/
│   ├── StandardRide.cs
│   ├── LuxuryRide.cs
│   └── SharedRide.cs
└── Program.cs

```

----------

#### **3️⃣ Solution: Use Factory Pattern**

The **Factory Pattern** centralizes the logic for object creation in a single location. This improves maintainability and makes adding new ride types simple and error-free.

----------

### **✅ Project Structure After Applying Clean Architecture:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideFactory.cs
│   ├── Services/
│   │   └── RideService.cs
├── Domain/
│   ├── Entities/
│   │   ├── IRide.cs
│   │   ├── StandardRide.cs
│   │   ├── LuxuryRide.cs
│   │   └── SharedRide.cs
├── Infrastructure/
│   ├── Factories/
│   │   └── RideFactory.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement Factory Pattern**

#### **Step 1: Define the Common Interface for Rides**

In the `Domain/Entities` folder:

```csharp
public interface IRide
{
    string GetRideDetails();
}

```

----------

#### **Step 2: Create Different Ride Types**

```csharp
public class StandardRide : IRide
{
    public string GetRideDetails()
    {
        return "Standard Ride";
    }
}

public class LuxuryRide : IRide
{
    public string GetRideDetails()
    {
        return "Luxury Ride";
    }
}

public class SharedRide : IRide
{
    public string GetRideDetails()
    {
        return "Shared Ride";
    }
}

```

----------

#### **Step 3: Create the Factory Interface**

In the `Application/Interfaces` folder:

```csharp
public interface IRideFactory
{
    IRide CreateRide(string rideType);
}

```

----------

#### **Step 4: Implement the Factory**

In the `Infrastructure/Factories` folder:

```csharp
public class RideFactory : IRideFactory
{
    public IRide CreateRide(string rideType)
    {
        return rideType switch
        {
            "Standard" => new StandardRide(),
            "Luxury" => new LuxuryRide(),
            "Shared" => new SharedRide(),
            _ => throw new ArgumentException("Invalid ride type!"),
        };
    }
}

```

----------

#### **Step 5: Use the Factory in the Service Layer**

```csharp
public class RideService
{
    private readonly IRideFactory _rideFactory;

    public RideService(IRideFactory rideFactory)
    {
        _rideFactory = rideFactory;
    }

    public IRide GetRide(string rideType)
    {
        return _rideFactory.CreateRide(rideType);
    }
}

```

----------

#### **Step 6: Create the Controller**

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

    [HttpGet("{rideType}")]
    public IActionResult GetRideDetails(string rideType)
    {
        var ride = _rideService.GetRide(rideType);
        return Ok(ride.GetRideDetails());
    }
}

```

----------

#### **Step 7: Register in Program.cs**

```csharp
builder.Services.AddScoped<IRideFactory, RideFactory>();
builder.Services.AddScoped<RideService>();

```

----------

### **8️⃣ Impact of Using Factory Pattern**

#### **Before Optimization:**

-   Repeated code for creating objects.
-   Hard to add new ride types without modifying multiple places.
-   Difficult to maintain and prone to errors.

#### **After Optimization:**

-   Centralized logic for creating objects in the factory.
-   Adding new types is simple and doesn’t affect existing code.
-   The code is cleaner, easier to maintain, and more testable.

----------


