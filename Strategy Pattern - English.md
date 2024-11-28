### Strategy Pattern 

----------

#### **1️⃣ What is the Strategy Pattern?**

The **Strategy Pattern** is a design pattern that allows you to define a family of algorithms or strategies, encapsulate each one in a separate class, and make them interchangeable.

-   The core idea is to select and execute the required strategy dynamically at runtime without modifying the existing code.

----------

### **Purpose of the Strategy Pattern**

1.  **Eliminate Repeated Code:** Remove multiple conditional statements.
2.  **Add New Strategies Easily:** Introduce new algorithms or logic without changing existing code.
3.  **Follow the Single Responsibility Principle (SRP):** Each algorithm is handled in its own class.

----------

### **2️⃣ Problem Before Using the Strategy Pattern**

In a ride-share application, there are multiple ways to calculate ride costs, such as:

1.  Cost based on **distance only**.
2.  Cost based on **distance and time**.
3.  Cost based on **car type**.

If all these strategies are implemented in a single method with conditions, the code will:

1.  **Become complicated and filled with `if/else` statements.**
2.  **Be difficult to maintain.**
3.  **Lack flexibility for adding new strategies.**

----------

#### 🚫 **Code Without Strategy Pattern (Code Without Pattern):**

```csharp
public class RideCostService
{
    public double CalculateCost(string rideType, double distance, double time)
    {
        if (rideType == "Standard")
        {
            return distance * 1.5; // Standard Ride Cost
        }
        else if (rideType == "Luxury")
        {
            return (distance * 3) + (time * 2); // Luxury Ride Cost
        }
        else if (rideType == "Shared")
        {
            return distance * 0.8; // Shared Ride Cost
        }
        else
        {
            throw new ArgumentException("Unknown ride type!");
        }
    }
}

```

-   **Problem with this approach:**
    1.  The code is cluttered with conditions (`if/else`).
    2.  Adding a new ride type requires modifying this method, risking bugs.
    3.  It violates the Open/Closed Principle since the method is not closed for modifications.

----------

### **❌ Project Structure Before Applying Strategy Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideCostService.cs
└── Program.cs

```

----------

### **3️⃣ Solution: Using the Strategy Pattern**

With the **Strategy Pattern**, each calculation method (strategy) is encapsulated in its own class, and a common interface is used to unify their implementation.  
The main service selects the appropriate strategy dynamically.

----------

### **✅ Project Structure After Applying Strategy Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostStrategy.cs
│   ├── Services/
│   │   └── RideCostService.cs
├── Domain/
│   ├── Strategies/
│   │   ├── StandardRideCostStrategy.cs
│   │   ├── LuxuryRideCostStrategy.cs
│   │   └── SharedRideCostStrategy.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement Strategy Pattern**

----------

#### **Step 1: Define the Strategy Interface**

In `Application/Interfaces`:

```csharp
public interface IRideCostStrategy
{
    double CalculateCost(double distance, double time);
}

```

----------

#### **Step 2: Implement the Strategies**

**Standard Ride Cost:**

```csharp
public class StandardRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return distance * 1.5;
    }
}

```

**Luxury Ride Cost:**

```csharp
public class LuxuryRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return (distance * 3) + (time * 2);
    }
}

```

**Shared Ride Cost:**

```csharp
public class SharedRideCostStrategy : IRideCostStrategy
{
    public double CalculateCost(double distance, double time)
    {
        return distance * 0.8;
    }
}

```

----------

#### **Step 3: Create the Service to Handle Strategies**

In `Application/Services`:

```csharp
public class RideCostService
{
    private readonly IRideCostStrategy _rideCostStrategy;

    public RideCostService(IRideCostStrategy rideCostStrategy)
    {
        _rideCostStrategy = rideCostStrategy;
    }

    public double CalculateRideCost(double distance, double time)
    {
        return _rideCostStrategy.CalculateCost(distance, time);
    }
}

```

----------

#### **Step 4: Implement the Controller**

In `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly IServiceProvider _serviceProvider;

    public RideController(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    [HttpGet("cost")]
    public IActionResult GetRideCost(string rideType, double distance, double time)
    {
        IRideCostStrategy strategy = rideType switch
        {
            "Standard" => _serviceProvider.GetService<StandardRideCostStrategy>(),
            "Luxury" => _serviceProvider.GetService<LuxuryRideCostStrategy>(),
            "Shared" => _serviceProvider.GetService<SharedRideCostStrategy>(),
            _ => throw new ArgumentException("Invalid ride type")
        };

        var rideCostService = new RideCostService(strategy);
        var cost = rideCostService.CalculateRideCost(distance, time);

        return Ok(cost);
    }
}

```

----------

#### **Step 5: Register the Strategies in `Program.cs`**

```csharp
builder.Services.AddScoped<StandardRideCostStrategy>();
builder.Services.AddScoped<LuxuryRideCostStrategy>();
builder.Services.AddScoped<SharedRideCostStrategy>();

```

----------

### **5️⃣ Impact of Using Strategy Pattern**

#### **Before Using Strategy Pattern:**

-   The code was cluttered with conditions and hard to maintain.
-   Adding new ride types required modifying the main method.
-   The code violated the Open/Closed Principle.

#### **After Using Strategy Pattern:**

-   Each strategy is isolated in its own class, making it easier to maintain.
-   Adding new strategies requires no changes to the existing code.
-   The code follows the Open/Closed Principle and is more flexible.


