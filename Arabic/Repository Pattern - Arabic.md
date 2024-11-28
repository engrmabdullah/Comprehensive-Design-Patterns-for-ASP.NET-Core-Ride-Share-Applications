### **تحسين إدارة البيانات باستخدام Repository Pattern في ASP.NET Core باستخدام Clean Architecture**

----------

#### **ما هو Repository Pattern؟**

**Repository Pattern** هو أحد أنماط التصميم (Design Patterns) التي تهدف إلى فصل منطق الوصول إلى البيانات (Data Access Logic) عن منطق الأعمال (Business Logic).  
يُمكّنك هذا النمط من:

1.  تقليل التكرار في الكود.
2.  عزل التفاعل مع البيانات (مثل قاعدة البيانات أو خدمة API).
3.  تسهيل صيانة الكود واختباره.

----------

### **❌ المشكلة: كود غير منظم للوصول إلى البيانات**

في تطبيقات مشاركة الركوب، يعتمد المطورون غالبًا على استدعاء قاعدة البيانات مباشرة من خلال `DbContext` في كل مكان في الكود. يؤدي هذا إلى:

1.  **تكرار الكود**: نفس الاستعلام يتم تنفيذه في أماكن متعددة.
2.  **صعوبة الصيانة**: إذا تغيرت طريقة تخزين البيانات، يجب تعديل الكود في كل مكان.
3.  **صعوبة الاختبار**: لا يمكن فصل الوصول إلى قاعدة البيانات بسهولة أثناء اختبار الوحدة.

#### 🚫 **الكود قبل استخدام Repository Pattern (Code Without Pattern):**



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

### **❌ هيكلة المشروع قبل تطبيق Clean Architecture:**



    RootFolder/
    ├── Data/
    │   ├── RideShareDbContext.cs
    │   └── Driver.cs
    ├── Services/
    │   └── DriverService.cs
    └── Program.cs 

----------

### **✅ الحل: استخدام Repository Pattern وClean Architecture**

#### **لماذا Clean Architecture؟**

تقسيم المشروع إلى طبقات مستقلة يساعد على:

1.  **سهولة الصيانة**: كل طبقة تحتوي على مسؤوليات محددة.
2.  **الاختبار**: يمكن اختبار كل طبقة بشكل منفصل.
3.  **المرونة**: تسهيل تغيير أي طبقة دون التأثير على الطبقات الأخرى.

----------

### **✅ هيكلة المشروع بعد تطبيق Clean Architecture:**


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

----------

### **الخطوات التفصيلية لتطبيق Repository Pattern:**

#### **1️⃣ تعريف الواجهة (Interface)**

في مجلد `Application/Interfaces`، نُعرّف واجهة عامة لإدارة البيانات:

csharp

Copy code

    public interface IRepository<T> where T : class
    {
        Task<IEnumerable<T>> GetAllAsync();
        Task<T> GetByIdAsync(int id);
        Task AddAsync(T entity);
        Task UpdateAsync(T entity);
        Task DeleteAsync(int id);
    }

----------

#### **2️⃣ تنفيذ Repository (Implementation)**

في مجلد `Infrastructure/Data/Repositories`، ننفذ الواجهة:



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

----------

#### **3️⃣ إنشاء الكيانات (Entities)**

في مجلد `Domain/Entities`، نُعرّف كيان `Driver`:



    public class Driver
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public bool IsAvailable { get; set; }
    }

----------

#### **4️⃣ إنشاء الخدمة (Service Layer)**

في مجلد `Application/Services`، نُنشئ خدمة لإدارة السائقين:



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

----------

#### **5️⃣ إنشاء Controller**

في مجلد `WebApi/Controllers`، نُنشئ `DriverController`:



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

----------

#### **6️⃣ التسجيل في Program.cs**

في ملف `Program.cs`، نسجل الـ Repository والخدمات:



    builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
    builder.Services.AddScoped<DriverService>();

----------

### **🚀 التأثير قبل وبعد تطبيق Repository Pattern:**

#### **قبل التحسين:**

-   **صعوبة الصيانة:** الاعتماد الكبير على `DbContext` يجعل التعديلات معقدة.
-   **تكرار الكود:** نفس الاستعلام مكرر في عدة أماكن.
-   **اختبارات ضعيفة:** لا يمكن اختبار منطق الأعمال بسهولة.

#### **بعد التحسين:**

-   **سهولة الصيانة:** تم عزل منطق الوصول إلى البيانات في طبقة مستقلة.
-   **اختبارات فعالة:** يمكن اختبار الخدمات بسهولة باستخدام Mocking.
-   **مرونة أكبر:** إمكانية استبدال قاعدة البيانات بخدمة API أو أي مصدر بيانات آخر دون تعديل كبير في الكود.
