### **Improving Data Access with Repository Pattern in ASP.NET Core Using Clean Architecture**

----------

### **What is Repository Pattern?**

The **Repository Pattern** is a design pattern used to separate data access logic from business logic.  
It provides a standardized way to interact with the data source, making the code more maintainable, testable, and flexible.

----------

### **❌ Problem: Unorganized Data Access Code**

In ride-share applications, developers often rely on direct database calls using `DbContext`. This leads to:

1.  **Code duplication**: The same queries are repeated across different places.
2.  **Maintenance difficulties**: If the data storage changes, the code must be updated everywhere.
3.  **Testing challenges**: It’s difficult to isolate the database logic for unit testing.

#### 🚫 **Code Without Repository Pattern:**

```csharp
public async Task<List<Driver>> GetAvailableDrivers()
{
    using (var context = new RideShareDbContext())
    {
        return await context.Drivers.Where(d => d.IsAvailable).ToListAsync();
    }
}

public async Task<Driver> GetDriverById(int id)
{
    using (var context = new RideShareDbContext())
    {
        return await context.Drivers.FirstOrDefaultAsync(d => d.Id == id);
    }
}

```

----------

### **❌ Project Structure Before Applying Clean Architecture:**

```plaintext
RootFolder/
├── Data/
│   ├── RideShareDbContext.cs
│   └── Driver.cs
├── Services/
│   └── DriverService.cs
└── Program.cs

```

----------

### **✅ Solution: Repository Pattern with Clean Architecture**

#### **Why Clean Architecture?**

Clean Architecture separates the project into independent layers, making it:

1.  **Maintainable**: Each layer has a single responsibility.
2.  **Testable**: Each layer can be tested independently.
3.  **Flexible**: Changing one layer (e.g., data access) doesn’t impact other layers.

----------

### **✅ Project Structure After Applying Clean Architecture:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   └── IRepository.cs
│   ├── Services/
│   │   └── DriverService.cs
├── Domain/
│   ├── Entities/
│   │   └── Driver.cs
├── Infrastructure/
│   ├── Data/
│   │   ├── RideShareDbContext.cs
│   │   ├── Repositories/
│   │   │   ├── Repository.cs
│   │   │   └── DriverRepository.cs
├── WebApi/
│   ├── Controllers/
│   │   └── DriverController.cs
└── Program.cs

```

----------

### **Steps to Implement Repository Pattern**

#### **1️⃣ Define the Interface**

In the `Application/Interfaces` folder, define a generic repository interface:

```csharp
public interface IRepository<T> where T : class
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

```

----------

#### **2️⃣ Implement the Repository**

In the `Infrastructure/Data/Repositories` folder, implement the interface:

```csharp
public class Repository<T> : IRepository<T> where T : class
{
    private readonly RideShareDbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(RideShareDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}

```

----------

#### **3️⃣ Create the Entities**

In the `Domain/Entities` folder, define the `Driver` entity:

```csharp
public class Driver
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool IsAvailable { get; set; }
}

```

----------

#### **4️⃣ Create the Service Layer**

In the `Application/Services` folder, create a service to manage drivers:

```csharp
public class DriverService
{
    private readonly IRepository<Driver> _driverRepository;

    public DriverService(IRepository<Driver> driverRepository)
    {
        _driverRepository = driverRepository;
    }

    public async Task<List<Driver>> GetAvailableDrivers()
    {
        return (await _driverRepository.GetAllAsync())
            .Where(d => d.IsAvailable)
            .ToList();
    }
}

```

----------

#### **5️⃣ Create the Controller**

In the `WebApi/Controllers` folder, create the `DriverController`:

```csharp
[ApiController]
[Route("api/[controller]")]
public class DriverController : ControllerBase
{
    private readonly DriverService _driverService;

    public DriverController(DriverService driverService)
    {
        _driverService = driverService;
    }

    [HttpGet("available")]
    public async Task<IActionResult> GetAvailableDrivers()
    {
        var drivers = await _driverService.GetAvailableDrivers();
        return Ok(drivers);
    }
}

```

----------

#### **6️⃣ Register the Repository and Services in `Program.cs`**

In `Program.cs`, register the repository and services using Dependency Injection:

```csharp
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<DriverService>();

```

----------

### **🚀 Impact of Using Repository Pattern with Clean Architecture**

#### **Before Optimization:**

-   **Unmaintainable Code:** Direct `DbContext` usage leads to scattered and repetitive queries.
-   **Poor Testing:** Difficult to isolate database logic for unit testing.
-   **Inflexibility:** Hard to change the data source.

#### **After Optimization:**

-   **Maintainable Code:** Data access is centralized in the repository layer.
-   **Effective Testing:** Services can be tested independently with mocking.
-   **Flexibility:** Easy to replace the database with another source like an external API.




