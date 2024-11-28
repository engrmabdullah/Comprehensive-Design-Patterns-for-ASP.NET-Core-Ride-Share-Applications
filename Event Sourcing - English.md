### Event Sourcing 

----------

#### **1️⃣ What is Event Sourcing?**

Event Sourcing is a design pattern where the state of an object is determined by a sequence of events, rather than storing the final state directly.

-   Instead of saving the current state of an object, we record all the changes (events) made to it.
-   These events are stored in sequential order and can be replayed to reconstruct the object's state.

----------

#### **2️⃣ Problem Before Using Event Sourcing**

In a Ride-Share application, consider the lifecycle of a ride request:

-   A user requests a ride.
-   A driver accepts the ride.
-   The ride begins.
-   The ride ends.

**The problem with traditional methods:**

1.  Each state change updates the database directly, overwriting the previous state.
2.  Historical data is lost, making it hard to track the lifecycle of a ride.
3.  Debugging issues becomes difficult because there's no record of what happened.
4.  Analysis of past events (e.g., why a ride was canceled) becomes challenging.

----------

#### 🚫 **Code Without Event Sourcing:**

```csharp
public class RideRequest
{
    public int Id { get; set; }
    public string Status { get; set; } // Pending, Accepted, InProgress, Completed
    public DateTime LastUpdated { get; set; }
}

```

-   Any operation updates the state directly:

```csharp
public void UpdateRideStatus(int rideId, string status)
{
    var ride = _dbContext.Rides.FirstOrDefault(r => r.Id == rideId);
    if (ride != null)
    {
        ride.Status = status;
        ride.LastUpdated = DateTime.UtcNow;
        _dbContext.SaveChanges();
    }
}

```

**Problems:**

1.  Historical events are lost.
2.  Debugging and auditing are impossible.
3.  It's hard to understand the sequence of operations that led to the current state.

----------

### **❌ Project Structure Before Event Sourcing**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── RideRequest.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **3️⃣ The Solution: Using Event Sourcing**

----------

#### **Key Idea:**

-   Instead of saving the final state of an object, record every event that modifies the object.
-   These events are stored chronologically.
-   The current state of the object is reconstructed by replaying all its events.

----------

### **✅ Project Structure After Event Sourcing**

```plaintext
RootFolder/
├── Application/
│   ├── Events/
│   │   ├── RideRequestedEvent.cs
│   │   ├── RideAcceptedEvent.cs
│   │   ├── RideStartedEvent.cs
│   │   └── RideCompletedEvent.cs
│   ├── Services/
│   │   └── EventStoreService.cs
│   ├── Models/
│       └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **4️⃣ Steps to Implement Event Sourcing**

----------

#### **Step 1: Define Events**

##### **Example: Ride Requested Event**

```csharp
public class RideRequestedEvent
{
    public int RideId { get; set; }
    public int UserId { get; set; }
    public DateTime Timestamp { get; set; }
}

```

##### **Example: Ride Accepted Event**

```csharp
public class RideAcceptedEvent
{
    public int RideId { get; set; }
    public int DriverId { get; set; }
    public DateTime Timestamp { get; set; }
}

```

----------

#### **Step 2: Create an Event Store to Save Events**

```csharp
public class EventStoreService
{
    private readonly List<object> _events = new List<object>();

    public void SaveEvent(object @event)
    {
        _events.Add(@event);
    }

    public IEnumerable<object> GetEvents()
    {
        return _events;
    }
}

```

**Explanation:**

-   `SaveEvent`: Adds an event to the event store.
-   `GetEvents`: Retrieves all stored events.

----------

#### **Step 3: Create a Ride Aggregate to Rebuild State**

```csharp
public class Ride
{
    public int Id { get; private set; }
    public string Status { get; private set; }

    public void Apply(RideRequestedEvent @event)
    {
        Id = @event.RideId;
        Status = "Requested";
    }

    public void Apply(RideAcceptedEvent @event)
    {
        Status = "Accepted";
    }

    public void Apply(RideStartedEvent @event)
    {
        Status = "InProgress";
    }

    public void Apply(RideCompletedEvent @event)
    {
        Status = "Completed";
    }
}

```

**Explanation:**

-   The `Ride` aggregate uses events to update its state instead of directly modifying properties.

----------

#### **Step 4: Update the Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    private readonly EventStoreService _eventStore;

    public RideController(EventStoreService eventStore)
    {
        _eventStore = eventStore;
    }

    [HttpPost("request")]
    public IActionResult RequestRide([FromBody] RideRequestedEvent rideRequested)
    {
        _eventStore.SaveEvent(rideRequested);
        return Ok("Ride requested.");
    }

    [HttpPost("accept")]
    public IActionResult AcceptRide([FromBody] RideAcceptedEvent rideAccepted)
    {
        _eventStore.SaveEvent(rideAccepted);
        return Ok("Ride accepted.");
    }

    [HttpGet("rebuild/{rideId}")]
    public IActionResult RebuildRide(int rideId)
    {
        var events = _eventStore.GetEvents();
        var ride = new Ride();
        foreach (var @event in events)
        {
            if (@event is RideRequestedEvent r && r.RideId == rideId) ride.Apply(r);
            if (@event is RideAcceptedEvent a && a.RideId == rideId) ride.Apply(a);
        }
        return Ok(ride);
    }
}

```

----------

#### **Step 5: Register the Event Store in `Program.cs`**

```csharp
builder.Services.AddSingleton<EventStoreService>();

```

----------

### **5️⃣ Benefits of Event Sourcing**

#### **Before:**

-   The final state overwrites the previous state, leading to data loss.
-   Difficult to debug or audit past actions.

#### **After:**

-   Every event is recorded chronologically.
-   The entire history of actions is preserved for debugging or auditing.
-   The current state can be rebuilt at any time by replaying events.

----------


