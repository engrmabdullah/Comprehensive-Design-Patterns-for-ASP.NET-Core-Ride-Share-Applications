### **نمط وحدة العمل (Unit of Work Pattern) في ASP.NET Core باستخدام Clean Architecture**

----------

#### **1️⃣ ما هو Unit of Work Pattern؟**

**Unit of Work Pattern** هو نمط تصميم (Design Pattern) يُستخدم لإدارة العمليات داخل قاعدة البيانات كوحدة واحدة.  
يضمن هذا النمط أن يتم تنفيذ جميع العمليات ضمن معاملة واحدة (Transaction)، بحيث يتم تطبيق جميع التغييرات أو إلغاؤها بالكامل، مما يضمن تناسق البيانات.

**فوائد النمط:**

1.  تقليل عدد استدعاءات `SaveChanges()`.
2.  ضمان التناسق عند وجود عمليات متعددة في معاملة واحدة.
3.  تقليل تكرار الكود المتعلق بالمعاملات.

----------

#### **2️⃣ المشكلة قبل استخدام Unit of Work Pattern**

عند استخدام المستودعات (Repositories) بشكل منفصل، تكون كل مستودع مسؤول عن استدعاء `SaveChanges()` الخاص به، مما يسبب:

1.  **تكرار استدعاء SaveChanges():** يؤدي إلى زيادة الحمل على قاعدة البيانات.
2.  **عدم التناسق:** إذا فشلت عملية واحدة، يمكن أن يتم تطبيق العمليات الأخرى.
3.  **تكرار الكود:** إدارة المعاملات بشكل منفصل داخل كل مستودع يؤدي إلى تكرار نفس المنطق.

----------

#### 🚫 **الكود قبل تطبيق Unit of Work Pattern (Code Without Pattern):**

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

----------

### **❌ هيكلة المشروع قبل تطبيق Clean Architecture:**

```plaintext
RootFolder/
├── Data/
│   ├── RideShareDbContext.cs
│   ├── DriverRepository.cs
│   ├── VehicleRepository.cs
└── Program.cs

```

----------

#### **3️⃣ الحل: استخدام Unit of Work Pattern**

**Unit of Work Pattern** يركز إدارة العمليات عبر جميع المستودعات من خلال معاملة واحدة، مما يضمن:

1.  تنفيذ جميع العمليات ضمن معاملة واحدة.
2.  تقليل التكرار وتحسين الصيانة.
3.  الحفاظ على تناسق البيانات.

----------

### **✅ هيكلة المشروع بعد تطبيق Clean Architecture:**

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

### **4️⃣ خطوات تطبيق Unit of Work Pattern**

#### **الخطوة 1: تعريف واجهة Unit of Work**

في مجلد `Application/Interfaces`:

```csharp
public interface IUnitOfWork : IDisposable
{
    IRepository<Driver> Drivers { get; }
    IRepository<Vehicle> Vehicles { get; }
    Task<int> CommitAsync();
}

```

----------

#### **الخطوة 2: تنفيذ Unit of Work**

في مجلد `Infrastructure/Data`:

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

#### **الخطوة 3: تعريف واجهة المستودع (Generic Repository Interface)**

في مجلد `Application/Interfaces`:

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

#### **الخطوة 4: تنفيذ المستودع (Repository Implementation)**

في مجلد `Infrastructure/Data/Repositories`:

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

#### **الخطوة 5: استخدام Unit of Work في طبقة الخدمة**

في مجلد `Application/Services`:

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

#### **الخطوة 6: إنشاء Controller**

في مجلد `WebApi/Controllers`:

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

#### **الخطوة 7: التسجيل في Program.cs**

```csharp
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<DriverService>();

```

----------

### **8️⃣ التأثير قبل وبعد استخدام Unit of Work Pattern**

#### **قبل التحسين:**

-   يتم استدعاء `SaveChanges()` بشكل متكرر في كل مستودع.
-   احتمالية حدوث عدم تناسق في البيانات عند فشل عملية واحدة.
-   تكرار الكود الخاص بإدارة المعاملات.

#### **بعد التحسين:**

-   يتم تنفيذ جميع العمليات داخل معاملة واحدة، مما يضمن التناسق.
-   تقليل تكرار الكود.
-   تحسين الصيانة عن طريق إدارة المعاملات في مكان مركزي.


