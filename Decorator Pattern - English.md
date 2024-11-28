### Decorator Pattern

----------

#### **1️⃣ What is the Decorator Pattern?**

Imagine ordering a cup of tea from a café.

-   Initially, you get plain tea.
-   Then, you can add sugar, milk, or other customizations.

Each addition enhances the tea without altering the original recipe.

The **Decorator Pattern** works the same way:

-   It allows you to add functionality to an object dynamically without modifying its core code.

----------

#### **2️⃣ The Problem Before Using Decorator Pattern**

In a Ride-Share application, consider a service like `RideCostCalculator` that calculates the base cost of a ride based on distance.

Now, you want to add features like:

1.  **Waiting time fees.**
2.  **Additional fees for extra passengers.**
3.  **Discounts.**

**The Problem:**  
If you add all these features in one class or method, the code becomes messy, hard to read, and violates the **Single Responsibility Principle (SRP).**

----------

#### 🚫 **Code Without Decorator Pattern:**

```csharp
public class RideCostCalculator
{
    public double CalculateBaseCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        double cost = distance * 2.5; // Base cost
        cost += waitingTime * 0.5; // Waiting time fee
        cost += passengerCount > 1 ? (passengerCount - 1) * 1.0 : 0; // Extra passengers
        cost -= discount; // Discount
        return cost;
    }
}

```

**Problems:**

1.  Hard to add new features (like peak hour surcharges) without modifying the method.
2.  Violates **SRP** by handling multiple responsibilities in one method.
3.  The code is less flexible and harder to test or maintain.

----------

### **❌ Project Structure Before Decorator Pattern:**

```plaintext
RootFolder/
├── Application/
│   └── Services/
│       └── RideCostCalculator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **3️⃣ The Solution: Using Decorator Pattern**

----------

#### **The Idea:**

-   Separate each feature (e.g., waiting time fee, discount) into its own **Decorator** class.
-   Each decorator adds functionality to the core class (`RideCostCalculator`) dynamically.

----------

### **✅ Project Structure After Decorator Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideCostCalculator.cs
│   ├── Services/
│   │   ├── BaseCostCalculator.cs
│   │   ├── WaitingTimeFeeDecorator.cs
│   │   ├── PassengerFeeDecorator.cs
│   │   └── DiscountDecorator.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ Steps to Implement Decorator Pattern**

----------

#### **Step 1: Create a Common Interface**

In the `Application/Interfaces` folder:

```csharp
public interface IRideCostCalculator
{
    double CalculateCost(double distance);
}

```

**Explanation:**

-   `IRideCostCalculator` provides a unified method `CalculateCost`.
-   Both the base calculator and decorators will implement this interface.

----------

#### **Step 2: Create the Core Class**

In the `Application/Services` folder:

```csharp
public class BaseCostCalculator : IRideCostCalculator
{
    public double CalculateCost(double distance)
    {
        return distance * 2.5; // Base cost per kilometer
    }
}

```

**Explanation:**

-   `BaseCostCalculator` calculates the base ride cost using the distance.
-   This class doesn’t handle any additional features.

----------

#### **Step 3: Create Decorators for Additional Features**

----------

**WaitingTimeFeeDecorator:**

```csharp
public class WaitingTimeFeeDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly double _waitingTime;

    public WaitingTimeFeeDecorator(IRideCostCalculator innerCalculator, double waitingTime)
    {
        _innerCalculator = innerCalculator;
        _waitingTime = waitingTime;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        return baseCost + (_waitingTime * 0.5); // Add waiting time fee
    }
}

```

----------

**PassengerFeeDecorator:**

```csharp
public class PassengerFeeDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly int _passengerCount;

    public PassengerFeeDecorator(IRideCostCalculator innerCalculator, int passengerCount)
    {
        _innerCalculator = innerCalculator;
        _passengerCount = passengerCount;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        var extraFee = _passengerCount > 1 ? (_passengerCount - 1) * 1.0 : 0;
        return baseCost + extraFee; // Add extra passenger fees
    }
}

```

----------

**DiscountDecorator:**

```csharp
public class DiscountDecorator : IRideCostCalculator
{
    private readonly IRideCostCalculator _innerCalculator;
    private readonly double _discount;

    public DiscountDecorator(IRideCostCalculator innerCalculator, double discount)
    {
        _innerCalculator = innerCalculator;
        _discount = discount;
    }

    public double CalculateCost(double distance)
    {
        var baseCost = _innerCalculator.CalculateCost(distance);
        return baseCost - _discount; // Apply discount
    }
}

```

----------

#### **Step 4: Use Decorators in RideService**

In the `Application/Services` folder:

```csharp
public class RideService
{
    public double CalculateRideCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        IRideCostCalculator calculator = new BaseCostCalculator();

        calculator = new WaitingTimeFeeDecorator(calculator, waitingTime);
        calculator = new PassengerFeeDecorator(calculator, passengerCount);
        calculator = new DiscountDecorator(calculator, discount);

        return calculator.CalculateCost(distance);
    }
}

```

**Explanation:**

-   Start with the core calculator (`BaseCostCalculator`).
-   Dynamically add features (waiting time, passenger fee, discount) using decorators.
-   This makes the code flexible and easy to extend.

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

    [HttpPost("calculate-cost")]
    public IActionResult CalculateCost(double distance, double waitingTime, int passengerCount, double discount)
    {
        var cost = _rideService.CalculateRideCost(distance, waitingTime, passengerCount, discount);
        return Ok($"Total ride cost: {cost} USD");
    }
}

```

----------

### **5️⃣ Impact of Decorator Pattern**

#### **Before:**

-   The code was tightly coupled, with all features handled in one method.
-   Adding new features required modifying the existing code.

#### **After:**

-   Each feature is handled in its own decorator, adhering to the **Single Responsibility Principle (SRP)**.
-   Adding new features is easy—just create a new decorator.
-   The code is more modular, flexible, and maintainable.

----------

