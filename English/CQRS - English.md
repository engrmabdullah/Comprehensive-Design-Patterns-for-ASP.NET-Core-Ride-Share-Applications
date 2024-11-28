### CQRS (Command Query Responsibility Segregation) 

----------

#### **1️⃣ What is CQRS?**

CQRS (**Command Query Responsibility Segregation**) is a design pattern that separates the responsibilities of:

-   **Commands:** Operations that modify data (Write Operations), such as creating a trip or updating driver status.
-   **Queries:** Operations that read data (Read Operations), such as retrieving available drivers or trip details.

**Key Idea:**

-   Divide responsibilities into distinct objects or handlers for better clarity, scalability, and maintainability.

----------

#### **2️⃣ Problem Before Using CQRS**

In a Ride-Share application, we may have a single service handling both:

1.  Retrieving available drivers.
2.  Updating driver status.
3.  Adding new trips.

**Problems:**

1.  **Code Coupling:** Business logic for reading and writing is intertwined, leading to cluttered code.
2.  **Difficult Maintenance:** Adding new functionality increases complexity.
3.  **Performance Issues:** Complex read queries may slow down write operations.

----------

#### 🚫 **Code Without CQRS:**

```csharp
public class DriverService
{
    private readonly List<Driver> _drivers;

    public DriverService(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    // Reading: Retrieve available drivers
    public List<Driver> GetAvailableDrivers()
    {
        return _drivers.Where(d => d.IsAvailable).ToList();
    }

    // Writing: Update driver status
    public void UpdateDriverStatus(int driverId, bool isAvailable)
    {
        var driver = _drivers.FirstOrDefault(d => d.Id == driverId);
        if (driver != null)
        {
            driver.IsAvailable = isAvailable;
        }
    }

    // Writing: Add a new trip
    public void AddTrip(Trip trip)
    {
        // Logic for adding a trip
    }
}

```

**Problems:**

1.  All operations (read and write) are mixed in a single class.
2.  Bugs in one part of the code can easily affect other parts.
3.  Difficult to optimize read and write operations independently.

----------

### **❌ Project Structure Before CQRS**

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

### **3️⃣ The Solution: Using CQRS**

----------

#### **Key Idea:**

-   Separate **commands** (write operations) from **queries** (read operations).
-   Each operation has its dedicated handler.

----------

### **✅ Project Structure After CQRS**

```plaintext
RootFolder/
├── Application/
│   ├── Commands/
│   │   ├── AddTripCommand.cs
│   │   ├── UpdateDriverStatusCommand.cs
│   ├── Queries/
│   │   └── GetAvailableDriversQuery.cs
│   ├── Handlers/
│   │   ├── AddTripCommandHandler.cs
│   │   ├── UpdateDriverStatusCommandHandler.cs
│   │   └── GetAvailableDriversQueryHandler.cs
│   ├── Models/
│       └── Driver.cs
└── WebApi/
    └── Controllers/
        └── DriverController.cs

```

----------

### **4️⃣ Steps to Implement CQRS**

----------

#### **Step 1: Create Command Objects**

##### **Example: Update Driver Status Command**

```csharp
public class UpdateDriverStatusCommand
{
    public int DriverId { get; set; }
    public bool IsAvailable { get; set; }
}

```

##### **Example: Add Trip Command**

```csharp
public class AddTripCommand
{
    public int DriverId { get; set; }
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public double Distance { get; set; }
}

```

----------

#### **Step 2: Create Query Objects**

##### **Example: Get Available Drivers Query**

```csharp
public class GetAvailableDriversQuery
{
    // Add query parameters if needed
}

```

----------

#### **Step 3: Create Handlers for Commands and Queries**

##### **Handler for Updating Driver Status**

```csharp
public class UpdateDriverStatusCommandHandler
{
    private readonly List<Driver> _drivers;

    public UpdateDriverStatusCommandHandler(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public void Handle(UpdateDriverStatusCommand command)
    {
        var driver = _drivers.FirstOrDefault(d => d.Id == command.DriverId);
        if (driver != null)
        {
            driver.IsAvailable = command.IsAvailable;
        }
    }
}

```

##### **Handler for Getting Available Drivers**

```csharp
public class GetAvailableDriversQueryHandler
{
    private readonly List<Driver> _drivers;

    public GetAvailableDriversQueryHandler(List<Driver> drivers)
    {
        _drivers = drivers;
    }

    public List<Driver> Handle(GetAvailableDriversQuery query)
    {
        return _drivers.Where(d => d.IsAvailable).ToList();
    }
}

```

----------

#### **Step 4: Update the Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly UpdateDriverStatusCommandHandler _updateHandler;
    private readonly GetAvailableDriversQueryHandler _queryHandler;

    public DriverController(
        UpdateDriverStatusCommandHandler updateHandler,
        GetAvailableDriversQueryHandler queryHandler)
    {
        _updateHandler = updateHandler;
        _queryHandler = queryHandler;
    }

    [HttpGet("available")]
    public IActionResult GetAvailableDrivers()
    {
        var query = new GetAvailableDriversQuery();
        var drivers = _queryHandler.Handle(query);
        return Ok(drivers);
    }

    [HttpPost("update-status")]
    public IActionResult UpdateDriverStatus([FromBody] UpdateDriverStatusCommand command)
    {
        _updateHandler.Handle(command);
        return Ok("Driver status updated.");
    }
}

```

----------

#### **Step 5: Register the Handlers in `Program.cs`**

```csharp
builder.Services.AddScoped<UpdateDriverStatusCommandHandler>();
builder.Services.AddScoped<GetAvailableDriversQueryHandler>();

```

----------

### **5️⃣ Benefits of CQRS**

#### **Before:**

-   A single class mixed read and write operations, leading to cluttered and unmaintainable code.
-   Performance optimizations were difficult due to intertwined responsibilities.

#### **After:**

-   Clear separation of concerns:
    -   **Commands** handle write operations.
    -   **Queries** handle read operations.
-   Easier to scale and maintain the application.
-   Improved performance as read and write operations can be optimized independently.

----------


