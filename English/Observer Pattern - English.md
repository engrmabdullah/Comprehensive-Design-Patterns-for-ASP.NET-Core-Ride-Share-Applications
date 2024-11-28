### Observer Pattern

----------

#### **1️⃣ What is the Observer Pattern?**

The **Observer Pattern** is a design pattern where:

1.  A **Subject** maintains a list of observers (subscribed objects).
2.  Whenever the **Subject**'s state changes, it automatically notifies all its observers.

Think of a WhatsApp group, where a message sent by one member is instantly received by all others in the group. In programming, the **Subject** is the group, and the **Observers** are the members.

----------

### **The Problem Before Using the Observer Pattern**

#### Old Code:

```csharp
public class RideService
{
    private readonly NotificationService _notificationService;
    private readonly DatabaseService _databaseService;

    public RideService(NotificationService notificationService, DatabaseService databaseService)
    {
        _notificationService = notificationService;
        _databaseService = databaseService;
    }

    public void UpdateRideStatus(int rideId, string status)
    {
        // Update ride status
        _databaseService.UpdateRideStatus(rideId, status);

        // Notify observers
        _notificationService.NotifyDriver(rideId, status);
        _notificationService.NotifyUser(rideId, status);
    }
}

```

**Issues:**

1.  `RideService` is directly responsible for updating the database and notifying users/drivers.
2.  Adding a new observer (e.g., logging updates) requires modifying the `RideService` code, violating the **Open/Closed Principle**.
3.  Code becomes tightly coupled and harder to maintain.

----------

### **❌ Project Structure Before Observer Pattern:**

```plaintext
RootFolder/
├── Services/
│   └── RideService.cs
├── Infrastructure/
│   ├── NotificationService.cs
│   ├── DatabaseService.cs
└── Program.cs

```

----------

### **3️⃣ The Solution: Using the Observer Pattern**

With the **Observer Pattern**, we:

1.  Create a **Subject** that notifies all observers whenever a state change occurs.
2.  Decouple observers so they can be added or removed independently without modifying the subject or service code.

----------

### **✅ Project Structure After Observer Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   ├── IObserver.cs
│   │   └── ISubject.cs
│   ├── Observers/
│   │   ├── DriverObserver.cs
│   │   ├── UserObserver.cs
│   │   └── LoggingObserver.cs
│   ├── Services/
│   │   └── RideSubject.cs
├── Domain/
│   ├── Entities/
│   │   └── Ride.cs
├── Infrastructure/
│   ├── NotificationService.cs
│   ├── DatabaseService.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement the Observer Pattern**

----------

#### **Step 1: Create the Observer Interface**

```csharp
public interface IObserver
{
    void Update(int rideId, string status);
}

```

**Explanation:**

-   The `IObserver` interface represents all observers.
-   The `Update` method will be called whenever the **Subject** notifies the observers.

----------

#### **Step 2: Create the Subject Interface**

```csharp
public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify(int rideId, string status);
}

```

**Explanation:**

1.  `Attach`: Adds an observer to the list.
2.  `Detach`: Removes an observer from the list.
3.  `Notify`: Notifies all observers about a state change.

----------

#### **Step 3: Implement the Subject**

```csharp
public class RideSubject : ISubject
{
    private readonly List<IObserver> _observers = new();

    public void Attach(IObserver observer)
    {
        _observers.Add(observer);
    }

    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }

    public void Notify(int rideId, string status)
    {
        foreach (var observer in _observers)
        {
            observer.Update(rideId, status);
        }
    }
}

```

**Explanation:**

1.  `_observers`: A list of all subscribed observers.
2.  `Attach`: Adds a new observer to the list.
3.  `Detach`: Removes an existing observer from the list.
4.  `Notify`: Loops through all observers and calls their `Update` method.

----------

#### **Step 4: Create Observers**

**DriverObserver:**

```csharp
public class DriverObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"Driver notified: Ride {rideId} is now {status}");
    }
}

```

**Explanation:**

-   Notifies the driver when the ride status changes.

----------

**UserObserver:**

```csharp
public class UserObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"User notified: Ride {rideId} is now {status}");
    }
}

```

**Explanation:**

-   Notifies the user about the updated ride status.

----------

**LoggingObserver:**

```csharp
public class LoggingObserver : IObserver
{
    public void Update(int rideId, string status)
    {
        Console.WriteLine($"Log: Ride {rideId} status changed to {status}");
    }
}

```

**Explanation:**

-   Logs the status change for auditing or debugging purposes.

----------

#### **Step 5: Use the Subject in the RideService**

```csharp
public class RideService
{
    private readonly RideSubject _rideSubject;

    public RideService(RideSubject rideSubject)
    {
        _rideSubject = rideSubject;
    }

    public void UpdateRideStatus(int rideId, string status)
    {
        // Notify all observers
        _rideSubject.Notify(rideId, status);
    }
}

```

**Explanation:**

-   The `RideService` no longer directly handles observers.
-   Instead, it uses the `RideSubject` to notify all observers when the ride status changes.

----------

#### **Step 6: Register Services in `Program.cs`**

```csharp
var rideSubject = new RideSubject();
rideSubject.Attach(new DriverObserver());
rideSubject.Attach(new UserObserver());
rideSubject.Attach(new LoggingObserver());

builder.Services.AddSingleton(rideSubject);
builder.Services.AddScoped<RideService>();

```

**Explanation:**

1.  `RideSubject` is registered as a singleton since all observers need to share the same subject.
2.  Observers are attached to the subject during application startup.
3.  `RideService` is registered for dependency injection.

----------

#### **Step 7: Create the Controller**

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

    [HttpPost("update-status")]
    public IActionResult UpdateRideStatus(int rideId, string status)
    {
        _rideService.UpdateRideStatus(rideId, status);
        return Ok("Ride status updated successfully!");
    }
}

```

**Explanation:**

1.  The controller receives the ride ID and status.
2.  It calls the `RideService`, which in turn uses the `RideSubject` to notify all observers.

----------

### **5️⃣ Impact of Using Observer Pattern**

#### **Before:**

-   `RideService` directly handled notifications and status updates.
-   Adding new observers required modifying existing code.

#### **After:**

-   `RideService` only triggers the `Notify` method.
-   New observers can be added without modifying the existing code.
-   Adheres to the **Open/Closed Principle**.

----------

