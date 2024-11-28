###  Template Method Pattern

----------

#### **What is the Template Method Pattern?**

The **Template Method Pattern** is used when you have a set of steps (a "template") that need to be performed in a specific sequence, but some of these steps might differ depending on the context. This pattern allows you to define the **common steps** in a base class while letting the subclasses handle the **specific steps**.

----------

#### **Real-Life Analogy**

Imagine you’re preparing a car for a trip. The general steps are:

1.  **Check oil.**
2.  **Refuel.**
3.  **Start the engine.**

If it's a **gasoline car**, you'll fill it with gasoline and start the combustion engine. If it’s an **electric car**, you’ll charge the battery and start the electric motor. The **steps are the same**, but the **implementation differs** based on the car type.

----------

#### **Problem Before Using Template Method Pattern**

Without the Template Method Pattern, you might end up with repetitive code like this:

```csharp
public class GasolineCar
{
    public void PrepareForTrip()
    {
        Console.WriteLine("Checking oil...");
        Console.WriteLine("Filling up gasoline...");
        Console.WriteLine("Starting the gasoline engine...");
    }
}

public class ElectricCar
{
    public void PrepareForTrip()
    {
        Console.WriteLine("Checking oil...");
        Console.WriteLine("Charging the battery...");
        Console.WriteLine("Starting the electric engine...");
    }
}

```

----------

#### **Why is This a Problem?**

1.  **Code Duplication**: Common steps like "Check oil" are repeated in each class.
2.  **Difficult to Maintain**: If you want to add a new step, you'll need to modify every class.
3.  **Breaks the Single Responsibility Principle (SRP)**: Each class is responsible for more than one concern.

----------

#### **Solution: Using Template Method Pattern**

The Template Method Pattern extracts the **common steps** into a base class and delegates the **custom behavior** to the subclasses through **abstract methods**.

----------

### **1. Project Structure**

```plaintext
RideShareApp/
├── Domain/
│   └── Entities/
│       └── Ride.cs
├── Application/
│   └── RidePreparation/
│       ├── RidePreparationTemplate.cs
│       ├── GasolineRidePreparation.cs
│       └── ElectricRidePreparation.cs
├── Infrastructure/
└── Program.cs

```

----------

### **2. Code Implementation**

#### **Step 1: Define the Template Class**

```csharp
public abstract class RidePreparationTemplate
{
    // Template method
    public void PrepareRide()
    {
        CheckOil();
        Refuel();
        StartEngine();
    }

    protected void CheckOil()
    {
        Console.WriteLine("Checking oil...");
    }

    // Abstract methods to be implemented by subclasses
    protected abstract void Refuel();
    protected abstract void StartEngine();
}

```

----------

#### **Step 2: Implement the GasolineRidePreparation Class**

```csharp
public class GasolineRidePreparation : RidePreparationTemplate
{
    protected override void Refuel()
    {
        Console.WriteLine("Filling up gasoline...");
    }

    protected override void StartEngine()
    {
        Console.WriteLine("Starting the gasoline engine...");
    }
}

```

----------

#### **Step 3: Implement the ElectricRidePreparation Class**

```csharp
public class ElectricRidePreparation : RidePreparationTemplate
{
    protected override void Refuel()
    {
        Console.WriteLine("Charging the battery...");
    }

    protected override void StartEngine()
    {
        Console.WriteLine("Starting the electric engine...");
    }
}

```

----------

#### **Step 4: Use the Pattern in Main**

```csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Preparing a gasoline car:");
        RidePreparationTemplate gasolineRide = new GasolineRidePreparation();
        gasolineRide.PrepareRide();

        Console.WriteLine("\nPreparing an electric car:");
        RidePreparationTemplate electricRide = new ElectricRidePreparation();
        electricRide.PrepareRide();
    }
}

```

----------

### **3. Output**

When you run the program:

```plaintext
Preparing a gasoline car:
Checking oil...
Filling up gasoline...
Starting the gasoline engine...

Preparing an electric car:
Checking oil...
Charging the battery...
Starting the electric engine...

```

----------

### **4. Advantages of Template Method Pattern**

1.  **Reusability**: Common steps are defined in one place (the base class), reducing duplication.
2.  **Flexibility**: Subclasses can change only the steps they need to customize.
3.  **Maintainability**: Adding or modifying steps requires changes only in the base class.
4.  **Single Responsibility Principle (SRP)**: Each subclass focuses only on the behavior it needs to implement.

----------

### **5. Real-World Use Cases**

1.  **E-Commerce Order Processing**:
    
    -   Common steps: Validate the order, check stock, calculate shipping costs.
    -   Variable step: The shipping method (standard, express).
2.  **Vehicle Preparation**:
    
    -   Common steps: Check tires, fill fluids.
    -   Variable step: Refueling or recharging.
3.  **Data Processing Pipelines**:
    
    -   Common steps: Read data, validate format.
    -   Variable step: Process data type (e.g., text or image analysis).

----------

### **Conclusion**

The Template Method Pattern is an excellent way to standardize workflows while allowing flexibility for specific steps. It ensures your code is easy to maintain, reusable, and adheres to the **Open/Closed Principle** by keeping common logic intact and extending specific logic only when needed.


