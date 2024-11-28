###  Command Pattern

----------

#### **1️⃣ What is the Command Pattern?**

Think of a restaurant scenario, my friend. When you want to order food, you tell the waiter, "I want a pizza," instead of directly talking to the chef and explaining how to make it. The waiter takes your "command" and ensures it is executed properly without you worrying about the details.

Similarly, the **Command Pattern** allows you to separate the request (Command) from its execution.

-   **Purpose:** To encapsulate each operation or action into a separate object, making the code easier to manage and maintain.

----------

### **2️⃣ Problem Before Using the Command Pattern**

In a **Ride-Share** application, operations like:

1.  Creating a new ride.
2.  Updating ride details.
3.  Canceling a ride.

These can result in tightly coupled code where the service directly interacts with multiple components.

----------

#### 🚫 **Code Before Using the Command Pattern:**

```csharp
public class RideService
{
    private readonly RideRepository _rideRepository;

    public RideService(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void CreateRide(Ride ride)
    {
        // Save ride to the database
        _rideRepository.Add(ride);
    }

    public void CancelRide(int rideId)
    {
        var ride = _rideRepository.GetById(rideId);
        ride.Status = "Cancelled";
        _rideRepository.Update(ride);
    }
}

```

----------

### **❌ Project Structure Before Command Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideService.cs
├── Repositories/
│   └── RideRepository.cs
└── Program.cs

```

-   **Issues:**
    1.  Services are tightly coupled with repositories.
    2.  All operations are directly managed in `RideService`.
    3.  Difficult to add or modify operations without impacting existing code.

----------

### **3️⃣ Solution: Using the Command Pattern**

The **Command Pattern** separates each operation into its own `Command` object, and these commands are handled by their respective `Handler`.

----------

### **✅ Project Structure After Command Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Commands/
│   │   ├── CreateRideCommand.cs
│   │   ├── CancelRideCommand.cs
│   ├── Handlers/
│   │   ├── CreateRideCommandHandler.cs
│   │   ├── CancelRideCommandHandler.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── Repositories/
│   │   └── RideRepository.cs
├── WebApi/
│   ├── Controllers/
│   │   └── RideController.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement the Command Pattern**

----------

#### **Step 1: Define the Commands**

**CreateRideCommand:**  
Encapsulates the data needed to create a ride.  
In `Application/Commands`:

```csharp
public class CreateRideCommand
{
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
}

```

**CancelRideCommand:**  
Encapsulates the data needed to cancel a ride.

```csharp
public class CancelRideCommand
{
    public int RideId { get; set; }
}

```

----------

#### **Step 2: Create Handlers**

Handlers are responsible for executing the logic of their respective commands.

**CreateRideCommandHandler:**  
In `Application/Handlers`:

```csharp
public class CreateRideCommandHandler
{
    private readonly RideRepository _rideRepository;

    public CreateRideCommandHandler(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void Handle(CreateRideCommand command)
    {
        var ride = new Ride
        {
            DriverId = command.DriverId,
            UserId = command.UserId,
            Distance = command.Distance,
            Status = "Created"
        };

        _rideRepository.Add(ride);
    }
}

```

**CancelRideCommandHandler:**

```csharp
public class CancelRideCommandHandler
{
    private readonly RideRepository _rideRepository;

    public CancelRideCommandHandler(RideRepository rideRepository)
    {
        _rideRepository = rideRepository;
    }

    public void Handle(CancelRideCommand command)
    {
        var ride = _rideRepository.GetById(command.RideId);
        if (ride != null)
        {
            ride.Status = "Cancelled";
            _rideRepository.Update(ride);
        }
    }
}

```

----------

#### **Step 3: Define the Entity**

In `Domain/Entities`:

```csharp
public class Ride
{
    public int Id { get; set; }
    public int DriverId { get; set; }
    public int UserId { get; set; }
    public double Distance { get; set; }
    public string Status { get; set; }
}

```

----------

#### **Step 4: Implement the Repository**

In `Infrastructure/Repositories`:

```csharp
public class RideRepository
{
    private readonly List<Ride> _rides = new();

    public void Add(Ride ride)
    {
        _rides.Add(ride);
    }

    public Ride GetById(int id)
    {
        return _rides.FirstOrDefault(r => r.Id == id);
    }

    public void Update(Ride ride)
    {
        var existingRide = GetById(ride.Id);
        if (existingRide != null)
        {
            existingRide.Status = ride.Status;
        }
    }
}

```

----------

#### **Step 5: Create the Controller**

In `WebApi/Controllers`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly CreateRideCommandHandler _createRideHandler;
    private readonly CancelRideCommandHandler _cancelRideHandler;

    public RideController(
        CreateRideCommandHandler createRideHandler,
        CancelRideCommandHandler cancelRideHandler)
    {
        _createRideHandler = createRideHandler;
        _cancelRideHandler = cancelRideHandler;
    }

    [HttpPost("create")]
    public IActionResult CreateRide([FromBody] CreateRideCommand command)
    {
        _createRideHandler.Handle(command);
        return Ok("Ride created successfully!");
    }

    [HttpPost("cancel")]
    public IActionResult CancelRide([FromBody] CancelRideCommand command)
    {
        _cancelRideHandler.Handle(command);
        return Ok("Ride cancelled successfully!");
    }
}

```

----------

#### **Step 6: Register Services in `Program.cs`**

```csharp
builder.Services.AddScoped<RideRepository>();
builder.Services.AddScoped<CreateRideCommandHandler>();
builder.Services.AddScoped<CancelRideCommandHandler>();

```

----------

### **5️⃣ Impact of Using the Command Pattern**

#### **Before:**

-   All operations were tightly coupled in `RideService`.
-   Code was complex and difficult to extend.
-   Adding new operations required modifying existing code.

#### **After:**

-   Each operation is encapsulated in its own `Command` and handled independently.
-   Code is more modular and maintainable.
-   Adding new operations does not affect existing code.

----------


