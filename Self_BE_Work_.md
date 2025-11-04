# .NET Developer Interview Q&A - 8 Years Experience

## 📋 Table of Contents
1. [C# Fundamentals](#c-fundamentals)
2. [ASP.NET Core/MVC](#aspnet-coremvc)
3. [Entity Framework & Database](#entity-framework--database)
4. [Design Patterns & Architecture](#design-patterns--architecture)
5. [Web API & RESTful Services](#web-api--restful-services)
6. [Security](#security)
7. [Performance & Optimization](#performance--optimization)
8. [Dependency Injection & IoC](#dependency-injection--ioc)
9. [Testing](#testing)
10. [Real-World Scenarios](#real-world-scenarios)

---

## C# Fundamentals

### Q1: Explain the difference between `abstract class` and `interface`. When would you use each?

**Answer:**
```csharp
// Abstract Class
public abstract class Animal 
{
    public abstract void MakeSound();
    public virtual void Eat() { /* Default implementation */ }
    protected int age; // Can have fields
}

// Interface
public interface IAnimal 
{
    void MakeSound(); // Only method signatures
    // Cannot have fields (before C# 8.0)
}
```

**Key Differences:**
- **Abstract Class:** Can have implementation, fields, constructors. Single inheritance.
- **Interface:** Only contracts, no implementation (before C# 8.0). Multiple inheritance. Default implementation in C# 8.0+.

**When to Use:**
- **Abstract Class:** When you have common base functionality for related classes (e.g., `Vehicle` → `Car`, `Truck`).
- **Interface:** When you need to define a contract that unrelated classes can implement (e.g., `IFlyable`, `ISwimmable`).

---

### Q2: What is the difference between `ref` and `out` parameters?

**Answer:**
```csharp
// ref - must be initialized before passing
int x = 10;
ModifyRef(ref x); // x must have a value

// out - doesn't need initialization
int y;
ModifyOut(out y); // y doesn't need initial value

void ModifyRef(ref int value) 
{
    value = 20; // Can read and modify
}

void ModifyOut(out int value) 
{
    value = 30; // Must assign before method ends
}
```

**Differences:**
- `ref`: Parameter must be initialized; can be read and modified.
- `out`: Parameter doesn't need initialization; must be assigned in method.

---

### Q3: Explain `async`/`await` and how it works under the hood.

**Answer:**
```csharp
// Async method
public async Task<string> FetchDataAsync() 
{
    var client = new HttpClient();
    var result = await client.GetStringAsync("https://api.example.com");
    return result;
}
```

**How it works:**
1. Method marked `async` returns `Task` or `Task<T>`.
2. `await` pauses execution, returns control to caller.
3. Execution resumes when awaited task completes.
4. Thread is freed during wait (non-blocking).

**Benefits:**
- Non-blocking I/O operations.
- Better scalability (threads not blocked).
- Improved user experience.

---

### Q4: What are `Generics` and why are they important?

**Answer:**
```csharp
// Generic class
public class Repository<T> where T : class
{
    private List<T> items = new List<T>();
    
    public void Add(T item) => items.Add(item);
    public T GetById(int id) => items.FirstOrDefault(x => x.Id == id);
}

// Generic method
public T Convert<T>(string value) where T : IConvertible
{
    return (T)Convert.ChangeType(value, typeof(T));
}
```

**Benefits:**
- Type safety at compile time.
- Code reusability.
- Performance (no boxing/unboxing).
- IntelliSense support.

---

## ASP.NET Core/MVC

### Q5: Explain the request pipeline in ASP.NET Core.

**Answer:**
```
Request → Middleware 1 → Middleware 2 → ... → Controller → Action → Response
```

**Pipeline Components:**
```csharp
public void Configure(IApplicationBuilder app) 
{
    app.UseRouting();           // Routing middleware
    app.UseAuthentication();    // Authentication
    app.UseAuthorization();     // Authorization
    app.UseEndpoints(...);      // Endpoints
}
```

**Middleware Order Matters:**
1. Exception handling (first)
2. Static files
3. Routing
4. Authentication
5. Authorization
6. Endpoints (last)

---

### Q6: What is Dependency Injection? Explain different service lifetimes.

**Answer:**
```csharp
// Registration in Program.cs or Startup.cs
services.AddScoped<IUserService, UserService>();      // Per request
services.AddTransient<IEmailService, EmailService>(); // New instance each time
services.AddSingleton<IConfiguration, Configuration>(); // Single instance
```

**Service Lifetimes:**
- **Transient:** New instance every time (lightweight services).
- **Scoped:** One instance per HTTP request (DB contexts, repositories).
- **Singleton:** Single instance for application lifetime (config, caching).

**Best Practices:**
- Use Scoped for DbContext.
- Use Transient for stateless services.
- Use Singleton carefully (thread safety required).

---

### Q7: Explain Model Binding and Validation in ASP.NET Core.

**Answer:**
```csharp
[HttpPost]
public IActionResult Create([FromBody] UserViewModel model) 
{
    if (!ModelState.IsValid) 
    {
        return BadRequest(ModelState);
    }
    // Process...
}

public class UserViewModel 
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }
    
    [Range(1, 100)]
    public int Age { get; set; }
}
```

**Model Binding Sources:**
- `[FromQuery]` - Query string
- `[FromBody]` - Request body
- `[FromRoute]` - Route parameters
- `[FromForm]` - Form data

---

### Q8: How do you handle errors and exceptions in ASP.NET Core?

**Answer:**
```csharp
// Global exception handler middleware
public class GlobalExceptionMiddleware 
{
    public async Task InvokeAsync(HttpContext context) 
    {
        try 
        {
            await _next(context);
        }
        catch (Exception ex) 
        {
            await HandleExceptionAsync(context, ex);
        }
    }
}

// Configure in Program.cs
app.UseExceptionHandler("/Error");
```

**Error Handling Strategies:**
- Global exception middleware
- Try-catch in controllers
- Result filters
- Custom error pages

---

## Entity Framework & Database

### Q9: Explain Entity Framework Core Code-First vs Database-First approach.

**Answer:**

**Code-First:**
```csharp
public class User 
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// Generate database from code
dbContext.Database.EnsureCreated();
```

**Database-First:**
```bash
# Scaffold from existing database
dotnet ef dbcontext scaffold "ConnectionString" Microsoft.EntityFrameworkCore.SqlServer
```

**When to Use:**
- **Code-First:** New projects, full control, migrations.
- **Database-First:** Existing database, legacy systems.

---

### Q10: What are Entity Framework Migrations? How do you manage them?

**Answer:**
```bash
# Create migration
dotnet ef migrations add InitialCreate

# Update database
dotnet ef database update

# Rollback
dotnet ef database update PreviousMigrationName
```

**Best Practices:**
- Version control migrations.
- Test migrations on staging.
- Create backups before applying.
- Use descriptive migration names.

---

### Q11: Explain LINQ and its different execution types.

**Answer:**
```csharp
// Deferred execution (IQueryable)
var query = context.Users.Where(u => u.Age > 18); // Not executed yet
var result = query.ToList(); // Executed here

// Immediate execution
var list = context.Users.Where(u => u.Age > 18).ToList(); // Executed immediately
```

**LINQ Types:**
- **IEnumerable:** In-memory queries (LINQ to Objects).
- **IQueryable:** Database queries (LINQ to SQL/EF).
- **Deferred:** Execution happens when enumerated.
- **Immediate:** Execution happens when called.

---

## Design Patterns & Architecture

### Q12: Explain Repository Pattern and Unit of Work.

**Answer:**
```csharp
// Repository
public interface IRepository<T> 
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Unit of Work
public interface IUnitOfWork 
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync();
}
```

**Benefits:**
- Separation of concerns.
- Testability (mock repositories).
- Centralized data access logic.

---

### Q13: What is CQRS? When would you use it?

**Answer:**
```csharp
// Command (Write)
public class CreateUserCommand 
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// Query (Read)
public class GetUserQuery 
{
    public int UserId { get; set; }
}

// Separate handlers
public class CreateUserCommandHandler { }
public class GetUserQueryHandler { }
```

**When to Use:**
- High read/write separation.
- Complex business logic.
- Different data models for read/write.
- Scalability requirements.

---

### Q14: Explain SOLID principles with examples.

**Answer:**

**S - Single Responsibility:**
```csharp
// Bad
public class User 
{
    public void Save() { }
    public void SendEmail() { }
    public void GenerateReport() { }
}

// Good
public class User { }
public class UserRepository { }
public class EmailService { }
```

**O - Open/Closed:**
```csharp
// Open for extension, closed for modification
public abstract class PaymentProcessor 
{
    public abstract void Process();
}
public class CreditCardPayment : PaymentProcessor { }
public class PayPalPayment : PaymentProcessor { }
```

**L - Liskov Substitution:**
```csharp
// Derived classes must be substitutable for base class
public class Bird { }
public class Sparrow : Bird { } // Can fly
public class Penguin : Bird { } // Cannot fly (violates LSP if Bird can fly)
```

**I - Interface Segregation:**
```csharp
// Small, focused interfaces
public interface IReadable { }
public interface IWritable { }
// Instead of one large interface
```

**D - Dependency Inversion:**
```csharp
// Depend on abstractions, not concretions
public class UserService 
{
    private readonly IRepository<User> _repository; // Abstraction
}
```

---

## Web API & RESTful Services

### Q15: Explain RESTful API principles and HTTP methods.

**Answer:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase 
{
    [HttpGet]           // GET /api/users
    public IActionResult GetAll() { }
    
    [HttpGet("{id}")]   // GET /api/users/1
    public IActionResult Get(int id) { }
    
    [HttpPost]         // POST /api/users
    public IActionResult Create([FromBody] User user) { }
    
    [HttpPut("{id}")]  // PUT /api/users/1
    public IActionResult Update(int id, [FromBody] User user) { }
    
    [HttpDelete("{id}")] // DELETE /api/users/1
    public IActionResult Delete(int id) { }
}
```

**REST Principles:**
- Stateless
- Resource-based URLs
- HTTP methods (GET, POST, PUT, DELETE)
- JSON/XML responses

---

### Q16: How do you implement versioning in Web API?

**Answer:**
```csharp
// URL versioning
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]")]

// Header versioning
[ApiVersion("2.0")]
[Route("api/[controller]")]
```

**Versioning Strategies:**
- URL versioning: `/api/v1/users`
- Header versioning: `api-version: 1.0`
- Query string: `?version=1.0`

---

## Security

### Q17: Explain authentication and authorization in ASP.NET Core.

**Answer:**
```csharp
// JWT Authentication
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => 
    {
        options.TokenValidationParameters = new TokenValidationParameters 
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            // ... other parameters
        };
    });

// Authorization
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly() { }

[Authorize(Policy = "RequireAge18")]
public IActionResult AgeRestricted() { }
```

**Types:**
- **Authentication:** Who you are (Login).
- **Authorization:** What you can do (Permissions).

---

### Q18: How do you prevent SQL Injection and XSS attacks?

**Answer:**

**SQL Injection Prevention:**
```csharp
// Bad
var sql = $"SELECT * FROM Users WHERE Name = '{userInput}'";

// Good - Parameterized queries
var sql = "SELECT * FROM Users WHERE Name = @name";
command.Parameters.AddWithValue("@name", userInput);

// EF Core (automatically safe)
context.Users.Where(u => u.Name == userInput);
```

**XSS Prevention:**
```csharp
// HTML encoding
@Html.Raw(userInput) // Dangerous
@userInput           // Safe (auto-encoded)

// Content Security Policy
app.Use(async (context, next) => 
{
    context.Response.Headers.Add("Content-Security-Policy", "default-src 'self'");
    await next();
});
```

---

## Performance & Optimization

### Q19: How do you optimize database queries in Entity Framework?

**Answer:**
```csharp
// Bad - N+1 problem
var users = context.Users.ToList();
foreach (var user in users) 
{
    var orders = context.Orders.Where(o => o.UserId == user.Id).ToList();
}

// Good - Eager loading
var users = context.Users
    .Include(u => u.Orders)
    .ToList();

// Good - Projection (select only needed fields)
var userNames = context.Users
    .Select(u => u.Name)
    .ToList();

// Good - Compiled queries (for repeated queries)
private static readonly Func<AppDbContext, int, Task<User>> GetUserById =
    EF.CompileAsyncQuery((AppDbContext db, int id) => 
        db.Users.FirstOrDefault(u => u.Id == id));
```

**Optimization Techniques:**
- Use `Include()` for related data.
- Use `Select()` for projections.
- Pagination with `Skip()` and `Take()`.
- Compiled queries for repeated operations.
- AsNoTracking() for read-only queries.

---

### Q20: Explain caching strategies in .NET applications.

**Answer:**
```csharp
// In-Memory Cache
services.AddMemoryCache();

// Distributed Cache (Redis)
services.AddStackExchangeRedisCache(options => 
{
    options.Configuration = "localhost:6379";
});

// Usage
[ResponseCache(Duration = 60)]
public IActionResult GetData() { }

// Programmatic caching
public class DataService 
{
    private readonly IMemoryCache _cache;
    
    public async Task<string> GetDataAsync(string key) 
    {
        if (!_cache.TryGetValue(key, out string cachedValue)) 
        {
            cachedValue = await FetchFromDatabase(key);
            _cache.Set(key, cachedValue, TimeSpan.FromMinutes(5));
        }
        return cachedValue;
    }
}
```

---

## Dependency Injection & IoC

### Q21: Explain Dependency Injection container and service lifetime management.

**Answer:**
```csharp
// Service registration
public void ConfigureServices(IServiceCollection services) 
{
    // Singleton - Single instance for app lifetime
    services.AddSingleton<IConfiguration>(provider => Configuration);
    
    // Scoped - One instance per HTTP request
    services.AddScoped<IUserService, UserService>();
    services.AddScoped<DbContext>(provider => 
        new AppDbContext(provider.GetService<IConfiguration>()));
    
    // Transient - New instance every time
    services.AddTransient<IEmailService, EmailService>();
    
    // Factory pattern
    services.AddScoped<IUserRepository>(provider => 
    {
        var dbContext = provider.GetService<DbContext>();
        return new UserRepository(dbContext);
    });
}
```

---

## Testing

### Q22: How do you write unit tests in .NET?

**Answer:**
```csharp
// Using xUnit
public class UserServiceTests 
{
    private readonly Mock<IUserRepository> _mockRepository;
    private readonly UserService _userService;
    
    public UserServiceTests() 
    {
        _mockRepository = new Mock<IUserRepository>();
        _userService = new UserService(_mockRepository.Object);
    }
    
    [Fact]
    public async Task GetUserById_ReturnsUser_WhenUserExists() 
    {
        // Arrange
        var userId = 1;
        var expectedUser = new User { Id = userId, Name = "John" };
        _mockRepository.Setup(r => r.GetByIdAsync(userId))
            .ReturnsAsync(expectedUser);
        
        // Act
        var result = await _userService.GetUserByIdAsync(userId);
        
        // Assert
        Assert.NotNull(result);
        Assert.Equal(expectedUser.Name, result.Name);
    }
}
```

---

## Real-World Scenarios

### Q23: How would you handle a high-traffic web application with database performance issues?

**Answer:**
1. **Caching Strategy:**
   - Redis for distributed caching
   - In-memory cache for frequently accessed data
   - Cache invalidation strategy

2. **Database Optimization:**
   - Index optimization
   - Query optimization (avoid N+1)
   - Connection pooling
   - Read replicas for read-heavy operations

3. **Application Level:**
   - Async/await for I/O operations
   - Response compression
   - CDN for static content
   - Load balancing

4. **Monitoring:**
   - Application Insights
   - Performance profiling
   - Database query analysis

---

### Q24: Explain a recent challenging project and how you solved a critical issue.

**Sample Answer:**
"In my last project, we faced OTP delivery delays during high traffic. The issue was synchronous email sending blocking request threads.

**Solution:**
1. Implemented background job processing with Hangfire
2. Moved email sending to async background tasks
3. Added retry mechanism with exponential backoff
4. Implemented email queue with Redis

**Results:**
- Response time reduced from 3s to 200ms
- 99.9% OTP delivery success rate
- Better scalability"

---

### Q25: How do you ensure code quality and maintainability?

**Answer:**
1. **Code Reviews:**
   - Peer reviews
   - Automated code analysis (SonarQube)

2. **Standards:**
   - Coding conventions
   - SOLID principles
   - Design patterns

3. **Testing:**
   - Unit tests (>80% coverage)
   - Integration tests
   - E2E tests for critical paths

4. **Documentation:**
   - XML comments
   - Architecture diagrams
   - README files

5. **CI/CD:**
   - Automated builds
   - Automated testing
   - Code quality gates

---

## Additional Tips for Interview

### Technical Discussion Points:
- **Architecture:** Microservices vs Monolith
- **Database:** SQL vs NoSQL, when to use each
- **Cloud:** Azure, AWS services
- **DevOps:** Docker, Kubernetes, CI/CD pipelines
- **Frontend:** Angular, React integration

### Questions to Ask:
1. What is the team structure?
2. What technologies are you currently using?
3. What are the biggest technical challenges?
4. How do you handle code reviews?
5. What is the deployment process?

### Key Strengths to Highlight:
- Problem-solving approach
- Experience with distributed systems
- Performance optimization
- Security best practices
- Team collaboration

---

## Quick Reference - Common Commands

```bash
# EF Core
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet ef dbcontext scaffold

# Build & Run
dotnet build
dotnet run
dotnet test

# NuGet
dotnet add package PackageName
dotnet restore
```

---

**Good Luck with your interview! 🚀**

