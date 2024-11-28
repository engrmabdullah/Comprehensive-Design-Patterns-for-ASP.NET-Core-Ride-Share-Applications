### Chain of Responsibility 

----------

#### **What is the Chain of Responsibility Pattern?**

Imagine you are working on a ride-share application like Uber, and you need to process a ride request in multiple stages, such as:

1.  Validating the ride request data.
2.  Checking for driver availability.
3.  Calculating the ride price.

Instead of writing a long and complex method to handle all these stages, we use the **Chain of Responsibility Pattern**. This pattern breaks down each responsibility into separate classes (called **Handlers**) and connects them in a chain. Each handler processes its part and then passes the request to the next handler.

----------

#### **The Problem Before Using the Pattern**

A typical implementation without the pattern might look like this:

```csharp
public void ProcessRideRequest(RideRequest request)
{
    // Validate request data
    if (string.IsNullOrEmpty(request.UserId) || string.IsNullOrEmpty(request.Destination))
    {
        Console.WriteLine("Invalid ride request.");
        return;
    }

    // Check driver availability
    if (!DriverService.IsDriverAvailable(request))
    {
        Console.WriteLine("No driver available.");
        return;
    }

    // Calculate price
    request.Price = PricingService.CalculatePrice(request);
    Console.WriteLine($"Ride request processed. Price: {request.Price}");
}

```

**Drawbacks:**

-   The code is large and hard to maintain.
-   Adding or removing stages is difficult.
-   Violates the **Single Responsibility Principle**.

----------

#### **The Solution Using Chain of Responsibility**

We will separate each responsibility into a class and chain them together.

----------

### **1. Project Structure (Root Folder)**

```plaintext
RideShareApp/
├── Domain/
│   ├── Entities/
│   │   └── RideRequest.cs
│   └── Interfaces/
│       └── IRequestHandler.cs
├── Application/
│   └── Handlers/
│       ├── ValidationHandler.cs
│       ├── DriverAvailabilityHandler.cs
│       └── PricingHandler.cs
└── Program.cs

```

----------

### **2. Code Implementation**

#### **Step 1: Define the RideRequest Class**

```csharp
public class RideRequest
{
    public string UserId { get; set; }
    public string Destination { get; set; }
    public decimal Price { get; set; }
}

```

----------

#### **Step 2: Define the IRequestHandler Interface**

```csharp
public interface IRequestHandler
{
    void SetNext(IRequestHandler nextHandler);
    void Handle(RideRequest request);
}

```

----------

#### **Step 3: Implement the Handlers**

##### **1. ValidationHandler**

```csharp
public class ValidationHandler : IRequestHandler
{
    private IRequestHandler _nextHandler;

    public void SetNext(IRequestHandler nextHandler)
    {
        _nextHandler = nextHandler;
    }

    public void Handle(RideRequest request)
    {
        if (string.IsNullOrEmpty(request.UserId) || string.IsNullOrEmpty(request.Destination))
        {
            Console.WriteLine("Validation failed: Missing user or destination.");
            return;
        }

        Console.WriteLine("Validation passed.");
        _nextHandler?.Handle(request);
    }
}

```

##### **2. DriverAvailabilityHandler**

```csharp
public class DriverAvailabilityHandler : IRequestHandler
{
    private IRequestHandler _nextHandler;

    public void SetNext(IRequestHandler nextHandler)
    {
        _nextHandler = nextHandler;
    }

    public void Handle(RideRequest request)
    {
        if (!DriverService.IsDriverAvailable(request))
        {
            Console.WriteLine("No driver available.");
            return;
        }

        Console.WriteLine("Driver is available.");
        _nextHandler?.Handle(request);
    }
}

```

##### **3. PricingHandler**

```csharp
public class PricingHandler : IRequestHandler
{
    public void SetNext(IRequestHandler nextHandler)
    {
        // Last handler in the chain
    }

    public void Handle(RideRequest request)
    {
        request.Price = PricingService.CalculatePrice(request);
        Console.WriteLine($"Ride request processed. Price: {request.Price}");
    }
}

```

----------

#### **Step 4: Setting up the Chain in Main**

```csharp
class Program
{
    static void Main(string[] args)
    {
        var rideRequest = new RideRequest
        {
            UserId = "123",
            Destination = "Downtown"
        };

        // Create the handlers
        var validationHandler = new ValidationHandler();
        var driverAvailabilityHandler = new DriverAvailabilityHandler();
        var pricingHandler = new PricingHandler();

        // Set up the chain
        validationHandler.SetNext(driverAvailabilityHandler);
        driverAvailabilityHandler.SetNext(pricingHandler);

        // Start processing the request
        validationHandler.Handle(rideRequest);
    }
}

```

----------

### **3. Output**

When you run the program:

```plaintext
Validation passed.
Driver is available.
Ride request processed. Price: 25.50

```

----------

### **4. Advantages of Using Chain of Responsibility**

1.  **Separation of Concerns**: Each handler is responsible for only one part of the logic.
2.  **Flexibility**: Adding or removing handlers is easy and does not affect other handlers.
3.  **Reusability**: Handlers can be reused in other workflows.
4.  **Extensibility**: New handlers can be added without modifying existing code.

----------

### **5. Real-Life Example**

Imagine you're placing an order online:

1.  **Validation**: Check if the user is logged in and has entered valid payment details.
2.  **Inventory Check**: Verify that the product is available in stock.
3.  **Payment Processing**: Deduct the amount from the user's account.

The **Chain of Responsibility** pattern is perfect for such scenarios.

----------


