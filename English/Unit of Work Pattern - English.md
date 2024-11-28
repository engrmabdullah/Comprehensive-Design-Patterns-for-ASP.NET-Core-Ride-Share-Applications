### **Unit of Work Pattern in ASP.NET Core Using Clean Architecture**

----------

#### **1️⃣ What is Unit of Work Pattern?**

The **Unit of Work Pattern** is a design pattern that manages transactions across multiple repositories. It ensures that a group of operations either succeeds or fails as a whole, maintaining data consistency.

This pattern is particularly useful when:

1.  Multiple database operations are involved in a single transaction.
2.  You need to reduce redundant `SaveChanges()` calls.

----------

#### **2️⃣ Problem Before Using Unit of Work Pattern**

In a typical setup, repositories handle database updates independently. This can cause issues such as:

1.  **Multiple SaveChanges() Calls**: Each repository calls `SaveChanges()` separately, increasing database overhead.
2.  **Transaction Inconsistency**: If one operation fails, changes from other operations may still be saved.
3.  **Code Duplication**: Managing transactions in multiple repositories results in redundant logic.

----------

#### 🚫 **Code Before Applying Unit of Work Pattern (Code Without Pattern):**

```csharp
public async Task CompleteDriverRegistration(Driver driver, Vehicle vehicle)
{
    await _driverRepository.AddAsync(driver);
    await _vehicleRepository.AddAsync(vehicle);

    // Each repository calls SaveChanges separately
    await _driverRepository.SaveChangesAsync();
    await _vehicleRepository.SaveChangesAsync();
}

```

### **❌ Project Structure Before Applying Clean Architecture:**

```plaintext
RootFolder/
├── Data/
│   ├── RideShareDbContext.cs
│   ├── DriverRepository.cs
│   ├── VehicleRepository.cs
└── Program.cs

```

----------

#### **3️⃣ Solution: Use Unit of Work Pattern**

The **Unit of Work Pattern** centralizes the transaction logic, ensuring that:

1.  Changes across multiple repositories are committed in a single transaction.
2.  Code duplication is minimized.
3.  Database consistency is maintained.

----------

### **✅ Project Structure After Applying Clean Architecture:**

```plaintext
RootFolder/
├── Application/
│   ├── Interfaces/
│   │   ├── IUnitOfWork.cs
│   │   ├── IRepository.cs
│   ├── Services/
│   │   └── DriverService.cs
├── Domain/
│   ├── Entities/
│   │   ├── Driver.cs
│   │   └── Vehicle.cs
├── Infrastructure/
│   ├── Data/
│   │   ├── RideShareDbContext.cs
│   │   ├── Repositories/
│   │   │   ├── Repository.cs
│   │   │   ├── DriverRepository.cs
│   │   │   └── VehicleRepository.cs
│   │   └── UnitOfWork.cs
├── WebApi/
│   ├── Controllers/
│   │   └── DriverController.cs
└── Program.cs

```

----------

### **4️⃣ Steps to Implement Unit of Work Pattern**

#### **Step 1: Define the Unit of Work Interface**

In the `Application/Interfaces` folder:

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<Driver> Drivers { get; }
    IRepository<Vehicle> Vehicles { get; }
    Task<int> CommitAsync();
}

```

----------

#### **Step 2: Implement the Unit of Work**

In the `Infrastructure/Data` folder:

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly RideShareDbContext _context;

    public IRepository<Driver> Drivers { get; }
    public IRepository<Vehicle> Vehicles { get; }

    public UnitOfWork(RideShareDbContext context, 
                      IRepository<Driver> driverRepository, 
                      IRepository<Vehicle> vehicleRepository)
    {
        _context = context;
        Drivers = driverRepository;
        Vehicles = vehicleRepository;
    }

    public async Task<int> CommitAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
    }
}

```

----------

#### **Step 3: Define Generic Repository Interface**

In the `Application/Interfaces` folder:

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

#### **Step 4: Implement the Generic Repository**

In the `Infrastructure/Data/Repositories` folder:

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
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
        }
    }
}

```

----------

#### **Step 5: Use Unit of Work in Service Layer**

In the `Application/Services` folder:

```csharp
public class DriverService
{
    private readonly IUnitOfWork _unitOfWork;

    public DriverService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task CompleteDriverRegistration(Driver driver, Vehicle vehicle)
    {
        await _unitOfWork.Drivers.AddAsync(driver);
        await _unitOfWork.Vehicles.AddAsync(vehicle);
        
        // Commit both operations in a single transaction
        await _unitOfWork.CommitAsync();
    }
}

```

----------

#### **Step 6: Create the Controller**

In the `WebApi/Controllers` folder:

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

    [HttpPost("register")]
    public async Task<IActionResult> CompleteDriverRegistration([FromBody] DriverRegistrationDto dto)
    {
        var driver = new Driver { Name = dto.DriverName };
        var vehicle = new Vehicle { Model = dto.VehicleModel };

        await _driverService.CompleteDriverRegistration(driver, vehicle);

        return Ok();
    }
}

```

----------

#### **Step 7: Register Unit of Work and Repositories in `Program.cs`**

```csharp
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<DriverService>();

```

----------

### **8️⃣ Impact of Using Unit of Work Pattern**

#### **Before Optimization:**

-   Multiple `SaveChanges()` calls for different repositories.
-   Increased risk of transaction inconsistencies.
-   Code duplication for managing transactions.

#### **After Optimization:**

-   Single transaction for multiple operations ensures consistency.
-   Reduced code duplication.
-   Centralized transaction management improves maintainability.


