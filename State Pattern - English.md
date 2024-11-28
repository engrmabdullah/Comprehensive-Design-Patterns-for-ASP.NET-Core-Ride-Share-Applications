###  State Pattern

----------

### **1️⃣ What is the State Pattern?**

The **State Pattern** allows an object to change its behavior dynamically based on its current state.  
Instead of writing multiple `if-else` or `switch-case` statements to handle state changes, each state is encapsulated in a separate class.  
The main object holds a reference to the current state and delegates state-specific behavior to that state.

----------

### **2️⃣ Problem Before Using the State Pattern**

In a Ride-Share application, the ride can go through various states such as:

-   **Requested**
-   **Accepted**
-   **Started**
-   **Completed**

In a traditional approach, you'd use conditional statements to check the current state and decide the behavior.

#### **Code Without State Pattern:**

```csharp
public class Ride
{
    public string Status { get; set; }

    public void Process()
    {
        if (Status == "Requested")
        {
            Console.WriteLine("Processing ride request...");
            Status = "Accepted";
        }
        else if (Status == "Accepted")
        {
            Console.WriteLine("Starting the ride...");
            Status = "Started";
        }
        else if (Status == "Started")
        {
            Console.WriteLine("Completing the ride...");
            Status = "Completed";
        }
        else
        {
            Console.WriteLine("Invalid state!");
        }
    }
}

```

----------

### **Problems in This Approach:**

1.  **Too Many Conditional Statements:**  
    The more states you add, the more complex the code becomes.
    
2.  **Violates Single Responsibility Principle (SRP):**  
    The `Ride` class handles both the state transitions and business logic, making it harder to maintain.
    
3.  **Difficult to Extend:**  
    Adding a new state means modifying the existing logic and adding more conditions.
    

----------

### **3️⃣ Solution: Using the State Pattern**

----------

#### **Key Idea:**

-   Each state is encapsulated in its own class.
-   The main object (e.g., `Ride`) delegates state-specific behavior to the current state object.
-   Transitioning between states is managed by changing the current state object.

----------

### **4️⃣ Project Structure**

#### **Before State Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

#### **After State Pattern:**

```plaintext
RootFolder/
├── Application/
│   ├── States/
│   │   ├── RideRequestedState.cs
│   │   ├── RideAcceptedState.cs
│   │   ├── RideStartedState.cs
│   │   └── RideCompletedState.cs
│   ├── Services/
│   │   └── RideService.cs
│   ├── Models/
│       └── Ride.cs
└── WebApi/
    └── Controllers/
        └── RideController.cs

```

----------

### **5️⃣ Code After Applying State Pattern**

----------

#### **Step 1: Define an Interface for States**

```csharp
public interface IRideState
{
    void Process(Ride ride);
}

```

----------

#### **Step 2: Implement Each State**

##### **RideRequestedState:**

```csharp
public class RideRequestedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Processing ride request...");
        ride.SetState(new RideAcceptedState());
    }
}

```

##### **RideAcceptedState:**

```csharp
public class RideAcceptedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Starting the ride...");
        ride.SetState(new RideStartedState());
    }
}

```

##### **RideStartedState:**

```csharp
public class RideStartedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Completing the ride...");
        ride.SetState(new RideCompletedState());
    }
}

```

##### **RideCompletedState:**

```csharp
public class RideCompletedState : IRideState
{
    public void Process(Ride ride)
    {
        Console.WriteLine("Ride is already completed.");
    }
}

```

----------

#### **Step 3: Update the Ride Class**

```csharp
public class Ride
{
    private IRideState _currentState;

    public Ride()
    {
        _currentState = new RideRequestedState(); // Initial State
    }

    public void SetState(IRideState state)
    {
        _currentState = state;
    }

    public void Process()
    {
        _currentState.Process(this);
    }
}

```

----------

#### **Step 4: Update the Controller**

```csharp
[ApiController]
[Route("api/[controller]")]
public class RideController : ControllerBase
{
    [HttpPost("process")]
    public IActionResult ProcessRide()
    {
        var ride = new Ride();
        ride.Process(); // Requested -> Accepted
        ride.Process(); // Accepted -> Started
        ride.Process(); // Started -> Completed
        ride.Process(); // Already Completed

        return Ok("Ride processing completed.");
    }
}

```

----------

### **6️⃣ Impact of Using State Pattern**

#### **Before:**

-   The code is cluttered with conditional statements.
-   The `Ride` class mixes logic for state transitions and state-specific behavior.

#### **After:**

-   Each state is encapsulated in its own class, improving readability and maintainability.
-   The `Ride` class only manages the current state, adhering to the Single Responsibility Principle (SRP).
-   Adding new states becomes straightforward—just add a new state class.

----------

### **7️⃣ Use Cases for the State Pattern**

1.  **Order Management Systems:**
    
    -   Track order states like `Pending`, `Shipped`, `Delivered`, `Cancelled`.
2.  **Ride-Share Applications:**
    
    -   Handle states like `Requested`, `Accepted`, `Started`, `Completed`.
3.  **Workflow Systems:**
    
    -   Manage task states such as `To Do`, `In Progress`, `Completed`.
4.  **Game Development:**
    
    -   Represent different states of a game character (e.g., `Idle`, `Running`, `Attacking`).
5.  **User Account Management:**
    
    -   States like `Active`, `Suspended`, `Deactivated`.

----------

