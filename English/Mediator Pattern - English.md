### Mediator Pattern 

----------

#### **1️⃣ What is the Mediator Pattern?**

The **Mediator Pattern** is a design pattern used to reduce complexity in communication between objects by introducing a central object called a "Mediator."

-   **Purpose:** To avoid direct interaction between objects, reducing tight coupling.
-   **Benefit:** Makes code more organized, easier to maintain, and less prone to errors.

----------

### **2️⃣ Problem Before Using the Mediator Pattern**

In a **Ride-Share** application, creating a new ride may involve multiple tasks, such as:

1.  **Sending a notification to the driver.**
2.  **Saving the ride details to the database.**
3.  **Notifying the user about the ride.**

#### **The Problem:**

-   If each object interacts directly with others:
    1.  **Increased coupling:** Objects become dependent on each other.
    2.  **Hard maintenance:** A change in one object requires changes in multiple places.
    3.  **Complex code:** Managing multiple interactions directly makes the code messy and error-prone.

----------

#### 🚫 **Code Before Using Mediator Pattern:**

```csharp
public class RideService
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideService()
    {
        _notificationService = new NotificationService();
        _databaseService = new DatabaseService();
    }

    public void CreateRide(Ride ride)
    {
        _databaseService.SaveRide(ride);
        _notificationService.NotifyDriver(ride.DriverId, ride);
        _notificationService.NotifyUser(ride.UserId, ride);
    }
}

```

**Problems with this code:**

1.  **Direct dependencies:** `RideService` is tightly coupled to `NotificationService` and `DatabaseService`.
2.  **Inflexibility:** Any change in these services requires changes in `RideService`.
3.  **Testing difficulty:** Mocking or stubbing dependent services becomes harder.

----------

### **❌ Project Structure Before Mediator Pattern:**

```plaintext
RootFolder/
├── Services/
│   ├── RideService.cs
│   ├── NotificationService.cs
│   └── DatabaseService.cs

```

----------

### **3️⃣ The Solution: Using the Mediator Pattern**

With the **Mediator Pattern**, a central mediator object manages interactions between different services.  
This eliminates the need for direct communication between objects.

----------

### **✅ Project Structure After Mediator Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRideRequestHandler.cs
│   ├── Mediators/
│   │   └── RideMediator.cs
│   ├── Notifications/
│   │   └── RideCreatedNotification.cs
│   ├── Commands/
│   │   └── CreateRideCommand.cs
│   ├── Services/
│   │   └── RideService.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Services/
│   │   ├── NotificationService.cs
│   │   └── DatabaseService.cs

```

----------

### **4️⃣ Steps to Implement the Mediator Pattern**

----------

#### **Step 1: Define the Command**

The `CreateRideCommand` is a request to create a new ride.  
In `Application/Commands`:

```csharp
public class CreateRideCommand
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
}

```

----------

#### **Step 2: Define the Notification**

The `RideCreatedNotification` is used to notify the driver and user about the ride.  
In `Application/Notifications`:

```csharp
public class RideCreatedNotification
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
}

```

----------

#### **Step 3: Create the Mediator**

The mediator handles all communication between services.  
In `Application/Mediators`:

```csharp
public class RideMediator
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideMediator(NotificationService notificationService, DatabaseService databaseService)
    {
        _notificationService = notificationService;
        _databaseService = databaseService;
    }

    public void Handle(CreateRideCommand command)
    {
        // Save ride details to the database
        _databaseService.SaveRide(new Ride
        {
            Id = command.RideId,
            DriverId = command.DriverId,
            UserId = command.UserId,
            Distance = command.Distance
        });

        // Send notifications
        _notificationService.NotifyDriver(command.DriverId, command);
        _notificationService.NotifyUser(command.UserId, command);
    }
}

```

----------

#### **Step 4: Implement Supporting Services**

**NotificationService:**  
Handles notifications for drivers and users.

```csharp
public class NotificationService
{
    public void NotifyDriver(int driverId, CreateRideCommand command)
    {
        Console.WriteLine($"Driver {driverId} notified for Ride {command.RideId}");
    }

    public void NotifyUser(int userId, CreateRideCommand command)
    {
        Console.WriteLine($"User {userId} notified for Ride {command.RideId}");
    }
}

```

**DatabaseService:**  
Handles saving ride data.

```csharp
public class DatabaseService
{
    public void SaveRide(Ride ride)
    {
        Console.WriteLine($"Ride {ride.Id} saved to the database.");
    }
}

```

----------

#### **Step 5: Use the Mediator in RideService**

In `Application/Services`:

```csharp
public class RideService
{
    private readonly RideMediator _rideMediator;

    public RideService(RideMediator rideMediator)
    {
        _rideMediator = rideMediator;
    }

    public void CreateRide(CreateRideCommand command)
    {
        _rideMediator.Handle(command);
    }
}

```

----------

#### **Step 6: Create the Controller**

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
    public IActionResult CreateRide([FromBody] CreateRideCommand command)
    {
        _rideService.CreateRide(command);
        return Ok("Ride created successfully!");
    }
}

```

----------

#### **Step 7: Register Mediator and Services in `Program.cs`**

```csharp
builder.Services.AddScoped<NotificationService>();
builder.Services.AddScoped<DatabaseService>();
builder.Services.AddScoped<RideMediator>();
builder.Services.AddScoped<RideService>();

```

----------

### **5️⃣ Impact of Using the Mediator Pattern**

#### **Before Mediator Pattern:**

-   Services were tightly coupled with one another.
-   Code was hard to maintain and extend.
-   Changes in one service often required changes in multiple places.

#### **After Mediator Pattern:**

-   Services communicate through the mediator.
-   Code is more modular and maintainable.
-   Reduces coupling and isolates responsibilities.

----------


