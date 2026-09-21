# C# / ASP.NET Core Interview Preparation Guide

## Simple, Step-by-Step Answers — Questions 1–65

> **Goal:** Prepare practical answers for a senior .NET / ASP.NET Core interview.  
> The explanations use simple language, but include enough technical detail for follow-up questions.

---

# 1. Write a Singleton class in C# to create a logger object

### Simple answer

A Singleton ensures that only **one instance** of a class is created and reused throughout the application.

For a logger, a Singleton can make sense because we normally do not need a separate logger object for every request.

### Recommended thread-safe implementation

```csharp
public sealed class Logger
{
    private static readonly Logger _instance = new Logger();

    private Logger()
    {
    }

    public static Logger Instance => _instance;

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

### Usage

```csharp
Logger.Instance.Log("Application started");
```

### How does this Singleton work?

- `private Logger()` prevents other classes from creating a `Logger` directly.
- `_instance` stores the single `Logger` object.
- `new Logger()` creates the object once when the `Logger` class is initialized.
- `Logger.Instance` returns the same object every time.
- Multiple calls do not create multiple `Logger` objects.

For example:

```csharp
Logger.Instance.Log("First message");
Logger.Instance.Log("Second message");
```

Both calls use the same instance:

```text
Call 1 ─┐
Call 2 ─┼──> Logger instance #1
Call 3 ─┘
```

### Interview point

In ASP.NET Core, I would generally prefer the **built-in DI container** rather than manually implementing Singleton.

```csharp
builder.Services.AddSingleton<ILoggerService, LoggerService>();
```

---

# 2. How can we optimize code for a multithreaded environment?

The main goal is to avoid unnecessary blocking and shared-state problems.

### Steps

**1. Avoid shared mutable state**

Bad — multiple threads increment the same field directly:

```csharp
private int _counter = 0;

public void Increment()
{
    _counter++; // not thread-safe, can lose updates
}
```

Better — give each thread its own state, or isolate the mutation:

```csharp
public int Increment(int currentValue)
{
    return currentValue + 1; // no shared field involved
}
```

**2. Prefer local variables**

```csharp
public int CalculateTotal(int[] items)
{
    int total = 0; // local to this call, safe across threads
    foreach (var item in items)
        total += item;

    return total;
}
```

**3. Use immutable objects where possible**

```csharp
public sealed record Money(decimal Amount, string Currency);

var price = new Money(100m, "USD");
// price.Amount = 200m; // not allowed, record is immutable
var discounted = price with { Amount = 90m }; // creates a new instance
```

**4. Use `async/await` for I/O operations**

```csharp
public async Task<Employee?> GetEmployeeAsync(int id)
{
    return await _db.Employees.FindAsync(id); // frees the thread while waiting on I/O
}
```

**5. Avoid unnecessary locks**

```csharp
// Unnecessary: locking around a read-only, already-thread-safe operation
lock (_lockObject)
{
    Console.WriteLine("Processing"); // no shared state involved, lock adds no value
}
```

**6. If shared state is required, use thread-safe collections**

```csharp
private static readonly ConcurrentDictionary<int, string> _cache = new();

public void AddToCache(int id, string value)
{
    _cache[id] = value; // safe for concurrent access without manual locking
}
```

**7. Use `lock` / `SemaphoreSlim` only where required**

```csharp
private static readonly SemaphoreSlim _semaphore = new(1, 1);

public async Task UpdateSharedFileAsync(string content)
{
    await _semaphore.WaitAsync();
    try
    {
        await File.WriteAllTextAsync("shared.txt", content); // async-friendly mutual exclusion
    }
    finally
    {
        _semaphore.Release();
    }
}
```

**8. Avoid blocking calls such as `.Result` and `.Wait()`**

```csharp
// Bad: blocks the calling thread, risks thread-pool starvation/deadlocks
var employee = _service.GetEmployeeAsync(id).Result;

// Better: propagate async all the way up
var employee = await _service.GetEmployeeAsync(id);
```

**9. Keep critical sections small**

```csharp
lock (_lockObject)
{
    _counter++; // only the actual shared-state mutation is inside the lock
}

LogAudit("Counter incremented"); // logging/IO kept outside the lock
```

**10. Use connection pooling and efficient database access**

```csharp
// Reuse the same connection string so ADO.NET/EF Core can pool connections
services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString)); // connection pooling is handled by the provider

public async Task<Employee?> GetAsync(int id)
{
    return await _db.Employees.AsNoTracking() // avoids unnecessary tracking overhead
        .FirstOrDefaultAsync(e => e.Id == id);
}
```

### Example

Bad:

```csharp
var result = GetDataAsync().Result;
```

Better:

```csharp
var result = await GetDataAsync();
```

### Thread-safe collection

```csharp
ConcurrentDictionary<int, string> users = new();
```

### Interview answer

> I optimize multithreaded code mainly by reducing shared mutable state, avoiding blocking calls, using async/await for I/O, using thread-safe collections where necessary, and keeping synchronization sections small.

---

# 3. When should Singleton pattern be avoided?

Avoid Singleton when the object:

- Contains request-specific state.
- Contains user-specific state.
- Is not thread-safe.
- Needs different instances in different contexts.
- Has dependencies that are scoped or transient.
- Makes unit testing difficult.
- Creates hidden global state.

### Example

A request context should normally **not** be Singleton.

```text
Request A → RequestContext A
Request B → RequestContext B
```

If it were Singleton:

```text
Request A ─┐
Request B ─┼→ Same object
Request C ─┘
```

This can cause data leaking between requests.

---

# 4. CRUD API using async/await with idempotency

Assume an Employee API.

## Model

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}
```

## Controller

```csharp
[ApiController]
[Route("api/employees")]
public class EmployeesController : ControllerBase
{
    private readonly IEmployeeService _service;

    public EmployeesController(IEmployeeService service)
    {
        _service = service;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        var employee = await _service.GetAsync(id);

        if (employee == null)
            return NotFound();

        return Ok(employee);
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        Employee employee,
        [FromHeader(Name = "Idempotency-Key")] string idempotencyKey)
    {
        var result = await _service.CreateAsync(employee, idempotencyKey);

        return CreatedAtAction(
            nameof(Get),
            new { id = result.Id },
            result);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> Update(
        int id,
        Employee employee)
    {
        var result = await _service.UpdateAsync(id, employee);

        if (result == null)
            return NotFound();

        return Ok(result);
    }

    [HttpPatch("{id}")]
    public async Task<IActionResult> Patch(int id, JsonPatchDocument<Employee> patch)
    {
        var result = await _service.PatchAsync(id, patch);

        if (result == null)
            return NotFound();

        return Ok(result);
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var deleted = await _service.DeleteAsync(id);

        if (!deleted)
            return NotFound();

        return NoContent();
    }
}
```

## What is idempotency?

An operation is idempotent when sending the **same request multiple times produces the same final result**.

For example:

```text
POST request
Idempotency-Key: abc123
```

The server stores:

```text
abc123 → Created Employee 101
```

If the client sends the same request again:

```text
abc123
```

The server returns the previous result instead of creating another employee.

### Important production point

The idempotency key should be stored in a durable store such as:

- Database
- Redis
- Distributed cache

For distributed applications, do not depend only on an in-memory dictionary.

---

# 5. Difference between POST, PUT and PATCH

| Method | Purpose                   | Typical behavior                     |
| ------ | ------------------------- | ------------------------------------ |
| POST   | Create/process            | Usually not idempotent               |
| PUT    | Replace/update resource   | Idempotent                           |
| PATCH  | Partially update resource | Usually intended for partial changes |

### POST

```http
POST /employees
```

Creates a new employee.

### PUT

```http
PUT /employees/10
```

Usually replaces the representation of employee 10.

```json
{
  "name": "John",
  "department": "IT"
}
```

Sending the same PUT repeatedly should leave the resource in the same final state.

### PATCH

```http
PATCH /employees/10
```

Updates only selected properties.

```json
{
  "department": "Finance"
}
```

---

# 6. Reverse a string without built-in reverse functions

Input:

```text
lineitem
```

Output:

```text
metienil
```

## Using a character array

```csharp
string input = "Hello";
string reversed = "";

for (int i = input.Length - 1; i >= 0; i--)
{
    reversed += input[i];
}

Console.WriteLine(reversed);
```

---

# 7. Top 3 most active users in a one-hour window

Assume:

```text
[Timestamp, UserID, Action, IPAddress]
```

## Approach

1. Parse each log.
2. Select logs inside the required one-hour window.
3. Group by UserID.
4. Count logs.
5. Order descending.
6. Take top 3.

```csharp
using System;
using System.Linq;
using System.Collections.Generic;

public class Log
{
    public DateTime Timestamp { get; set; }
    public int UserId { get; set; }
    public string Action { get; set; }
    public string IPAddress { get; set; }
}

public class Program
{
    public static void Main()
    {
        // Current time
        var now = DateTime.Now;

        // Sample logs
        var logs = new List<Log>
        {
            new Log { Timestamp = now.AddMinutes(-10), UserId = 101, Action = "Login",  IPAddress = "10.0.0.1" },
            new Log { Timestamp = now.AddMinutes(-20), UserId = 101, Action = "Search", IPAddress = "10.0.0.1" },
            new Log { Timestamp = now.AddMinutes(-30), UserId = 101, Action = "View",   IPAddress = "10.0.0.1" },
            new Log { Timestamp = now.AddMinutes(-40), UserId = 101, Action = "Logout", IPAddress = "10.0.0.1" },

            new Log { Timestamp = now.AddMinutes(-5),  UserId = 102, Action = "Login",  IPAddress = "10.0.0.2" },
            new Log { Timestamp = now.AddMinutes(-15), UserId = 102, Action = "Search", IPAddress = "10.0.0.2" },
            new Log { Timestamp = now.AddMinutes(-25), UserId = 102, Action = "View",   IPAddress = "10.0.0.2" },

            new Log { Timestamp = now.AddMinutes(-8),  UserId = 103, Action = "Login",  IPAddress = "10.0.0.3" },
            new Log { Timestamp = now.AddMinutes(-18), UserId = 103, Action = "Search", IPAddress = "10.0.0.3" },

            new Log { Timestamp = now.AddMinutes(-12), UserId = 104, Action = "Login",  IPAddress = "10.0.0.4" },

            // Older than 1 hour - should NOT be counted
            new Log { Timestamp = now.AddHours(-2), UserId = 105, Action = "Login", IPAddress = "10.0.0.5" }
        };

        // Last 1 hour
        var oneHourAgo = now.AddHours(-1);

        var result = logs
            .Where(x => x.Timestamp >= oneHourAgo &&
				   x.Timestamp <= now)
			.GroupBy(x=> x.UserId)
			.Select( g => new
					 {
						 UserId = g.Key,
						 Count = g.Count()
					 })
				.OrderByDescending(o=>o.Count)
				.Take(3)
					.ToList();

        // Display result
        Console.WriteLine("Top 3 Most Active Users - Last 1 Hour");
        Console.WriteLine("--------------------------------------");

        foreach (var user in result)
        {
            Console.WriteLine(
                $"UserId: {user.UserId}, Activities: {user.Count}");
        }
    }
}
```

### Complexity

Approximately:

```text
O(n log n)
```

because of sorting.

For very large log volumes, a frequency dictionary can reduce unnecessary sorting.

---

# 8. Find the most frequent user pair

Requirement:

> Two users logged in from the same IP address within 5 minutes of each other.

## Approach

1. Filter `Login` events.
2. Group by IP address.
3. Sort each group's events by timestamp.
4. Compare nearby events.
5. If the users are different and time difference <= 5 minutes, count the pair.
6. Return the pair with the highest count.

```csharp
var pairCounts = new Dictionary<(string User1, string User2), int>();

foreach (var ipGroup in loginLogs.GroupBy(x => x.IPAddress))
{
    var events = ipGroup
        .OrderBy(x => x.Timestamp)
        .ToList();

    for (int i = 0; i < events.Count; i++)
    {
        for (int j = i + 1; j < events.Count; j++)
        {
            if (events[j].Timestamp - events[i].Timestamp >
                TimeSpan.FromMinutes(5))
                break;

            if (events[i].UserId == events[j].UserId)
                continue;

            var pair = string.CompareOrdinal(
                events[i].UserId,
                events[j].UserId) < 0
                ? (events[i].UserId, events[j].UserId)
                : (events[j].UserId, events[i].UserId);

            pairCounts[pair] =
                pairCounts.GetValueOrDefault(pair) + 1;
        }
    }
}

var mostFrequentPair = pairCounts
    .OrderByDescending(x => x.Value)
    .FirstOrDefault();
```

### Important detail

Normalizing the pair:

```text
(A, B)
(B, A)
```

into:

```text
(A, B)
```

prevents counting the same pair twice.

---

# 9. Equals() vs GetHashCode()

## Equals()

Determines whether two objects are logically equal.

```csharp
a.Equals(b)
```

returns:

```text
true / false
```

## GetHashCode()

Returns an integer hash representing the object.

```csharp
int hash = obj.GetHashCode();
```

It is heavily used by:

- Dictionary
- HashSet
- Hashtable

### Relationship

If:

```csharp
a.Equals(b) == true
```

then:

```text
a.GetHashCode() == b.GetHashCode()
```

must also be true.

But the reverse is **not guaranteed**.

Two different objects can have the same hash code. This is called a hash collision.

### Object class

`System.Object` provides virtual methods such as:

```csharp
Equals()
GetHashCode()
ToString()
GetType()
```

When creating value-like domain classes, you may override `Equals()` and `GetHashCode()` together.

---

# 10. Dependency Injection in .NET

DI means:

> A class receives the objects it depends on instead of creating those objects itself.

## Without DI

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repository =
        new SqlOrderRepository();
}
```

The class is tightly coupled.

## With DI

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
}
```

Register:

```csharp
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
```

### What happens internally?

Conceptually:

```text
Application starts
      ↓
Service registrations are added
      ↓
DI container builds service information
      ↓
Controller requested
      ↓
Container checks controller dependencies
      ↓
Creates required objects
      ↓
Injects dependencies
      ↓
Controller executes
```

The built-in container handles:

- Object creation
- Dependency resolution
- Lifetimes
- Disposal

---

# 11. .NET memory: Stack, Heap and GC generations

## Stack

The stack is used for method execution and local execution state.

Example:

```csharp
void Calculate()
{
    int x = 10;
}
```

The method has stack-related execution data.

## Heap

Objects created with `new` are generally allocated on the managed heap.

```csharp
var employee = new Employee();
```

The object is managed by the GC.

### Important clarification

It is too simplistic to say:

> Value types are always stack and reference types are always heap.

Actual memory placement depends on context, lifetime, JIT/runtime optimizations, boxing, fields, closures, etc.

## Garbage Collector

The GC automatically identifies objects that are no longer reachable and reclaims their memory.

### Generations

```text
Gen 0 → short-lived objects
Gen 1 → intermediate objects
Gen 2 → long-lived objects
```

Example:

```csharp
var temporary = new object();
```

Short-lived objects are usually collected quickly.

Objects that survive collections can be promoted:

```text
Gen 0
  ↓ survives
Gen 1
  ↓ survives
Gen 2
```

### Why generations?

Most objects die young.

Therefore the GC can focus frequently on younger generations rather than scanning everything every time.

---

# 12. How do you design C# classes to be testable?

### Follow these principles

1. Depend on interfaces.
2. Use constructor injection.
3. Keep classes focused.
4. Avoid static dependencies.
5. Avoid direct database access inside business logic.
6. Avoid `new` for external dependencies inside the class.
7. Keep methods small.
8. Separate business logic from infrastructure.

# 12. How do you design C# classes to be testable?

For a **Senior .NET interview**, the main idea is:

> **A testable class should contain business logic that can be executed independently, without requiring a real database, API, file system, or other external dependency.**

Let's understand each principle with one practical example.

---

## 1. Depend on interfaces

❌ Tightly coupled:

```csharp
public class OrderService
{
    private readonly EmailService _emailService;

    public OrderService(EmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Better:

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

Now during testing:

```csharp
var mockEmailService = new Mock<IEmailService>();
```

You don't need the real email service.

---

# 2. Use Constructor Injection

Dependencies should be explicitly provided through the constructor.

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;

    public OrderService(
        IOrderRepository repository,
        IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
}
```

This makes dependencies obvious.

The test can provide mocks:

```csharp
var repository = new Mock<IOrderRepository>();
var emailService = new Mock<IEmailService>();

var service = new OrderService(
    repository.Object,
    emailService.Object);
```

---

# 3. Keep classes focused

A class should have a clear responsibility.

❌ Bad:

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        // Validate order

        // Save to database

        // Send email

        // Generate PDF

        // Upload PDF to Azure Blob

        // Write log
    }
}
```

This class is doing too many things.

Better:

```text
OrderService
    ↓
OrderRepository
EmailService
InvoiceService
BlobStorageService
```

Each responsibility can be tested independently.

This follows **Single Responsibility Principle (SRP)**.

---

# 4. Avoid static dependencies

Static dependencies are difficult to replace during unit testing.

For example:

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        var date = DateTime.Now;
    }
}
```

The test cannot easily control what `DateTime.Now` returns.

Instead:

```csharp
public interface IDateTimeProvider
{
    DateTime Now { get; }
}
```

Implementation:

```csharp
public class DateTimeProvider : IDateTimeProvider
{
    public DateTime Now => DateTime.Now;
}
```

Then:

```csharp
public class OrderService
{
    private readonly IDateTimeProvider _dateTime;

    public OrderService(IDateTimeProvider dateTime)
    {
        _dateTime = dateTime;
    }
}
```

Test:

```csharp
var mockDateTime = new Mock<IDateTimeProvider>();

mockDateTime
    .Setup(x => x.Now)
    .Returns(new DateTime(2026, 1, 1));
```

Now the test controls the date.

---

# 5. Avoid direct database access inside business logic

❌ Bad:

```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
        using var connection = new SqlConnection(connectionString);

        // SQL query
        // Insert into database
    }
}
```

Now testing `OrderService` requires a database.

Better:

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public async Task CreateOrder(Order order)
    {
        // Business logic

        await _repository.AddAsync(order);
    }
}
```

During testing:

```csharp
var repository = new Mock<IOrderRepository>();
```

No real database is required.

---

# 6. Avoid `new` for external dependencies

This is another important one.

❌ Bad:

```csharp
public class OrderService
{
    public void CreateOrder()
    {
        var emailService = new EmailService();

        emailService.Send();
    }
}
```

Why is this bad?

Because the class controls the dependency's creation.

You can't easily replace it with a mock.

Better:

```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

### Important distinction

Using `new` isn't inherently bad.

This is perfectly fine:

```csharp
var order = new Order();
```

The problem is using `new` to create **external dependencies** inside business logic.

For example:

```text
❌ new HttpClient()
❌ new SqlConnection()
❌ new EmailService()
❌ new BlobStorageService()
```

These should generally be abstracted/injected.

---

# 7. Keep methods small

Compare:

```csharp
public async Task ProcessOrder(Order order)
{
    // 200 lines
}
```

versus:

```csharp
public async Task ProcessOrder(Order order)
{
    ValidateOrder(order);
    CalculateTotal(order);
    await SaveOrder(order);
    await SendConfirmation(order);
}
```

Now each piece can be tested separately.

For example:

```csharp
[Fact]
public void CalculateTotal_ShouldIncludeTax()
{
    // Arrange
    // Act
    // Assert
}
```

Small methods also make failures easier to understand.

---

# 8. Separate business logic from infrastructure

This is one of the most important concepts.

### Infrastructure

Things that communicate with the outside world:

```text
SQL Server
Azure Blob
Service Bus
HTTP APIs
Email
File system
Key Vault
```

### Business logic

Your actual application rules:

```text
Is claim eligible?
Calculate premium
Validate order
Determine discount
Check claim status
```

Ideally:

```text
                OrderService
               Business Logic
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
    Repository   EmailService  API Client
          ↓          ↓          ↓
       SQL DB      Email       External API
```

The business layer doesn't need to know the implementation details.

---

# Putting everything together

Imagine your real application:

```csharp
public class ClaimService : IClaimService
{
    private readonly IClaimRepository _repository;
    private readonly IClaimsApiClient _apiClient;
    private readonly INotificationService _notificationService;

    public ClaimService(
        IClaimRepository repository,
        IClaimsApiClient apiClient,
        INotificationService notificationService)
    {
        _repository = repository;
        _apiClient = apiClient;
        _notificationService = notificationService;
    }

    public async Task ProcessClaim(int claimId)
    {
        var claim = await _repository.GetAsync(claimId);

        if (claim == null)
            throw new InvalidOperationException("Claim not found");

        var result = await _apiClient.ValidateAsync(claim);

        if (result.IsApproved)
        {
            claim.Status = "Approved";

            await _repository.UpdateAsync(claim);

            await _notificationService.NotifyAsync(claim);
        }
    }
}
```

Unit test:

```csharp
var repository = new Mock<IClaimRepository>();
var apiClient = new Mock<IClaimsApiClient>();
var notification = new Mock<INotificationService>();

var service = new ClaimService(
    repository.Object,
    apiClient.Object,
    notification.Object);
```

Now you can test:

```text
ClaimService
     │
     ├── Mock Repository
     ├── Mock API Client
     └── Mock Notification
```

**No SQL Server.
No real API.
No email.
No Azure dependency.**

That's what makes the class highly testable.

---

# 🎯 Senior Interview Answer

If the interviewer asks:

**"How do you design C# classes to be testable?"**

A strong answer would be:

> "I design classes with loose coupling and clear responsibilities. I depend on interfaces rather than concrete implementations and use constructor injection to provide dependencies. I avoid creating infrastructure dependencies directly inside the class and avoid static dependencies where they make behavior difficult to control in tests. I keep business logic separate from infrastructure such as databases, external APIs, Service Bus, and file storage. This allows me to mock external dependencies and unit-test the business logic in isolation. I also follow SRP and keep methods focused, which makes the code easier to test and maintain."

### One-line mental model

```text
Testable Class
     =
Business Logic
     +
Injected Dependencies
     +
Small Responsibilities
     -
Direct Infrastructure Dependencies
```

### 🔥 Interview follow-up you should prepare

**"What is the difference between a Unit Test and an Integration Test in .NET, and when would you use Moq?"**

---

# 13. How to identify design issues in C# code

When given code in an interview, check these areas:

### 1. SOLID violations

- Too many responsibilities
- Tight coupling
- Difficult extension
- Large interfaces

### 2. Async problems

Look for:

```csharp
.Result
.Wait()
```

### 3. Exception handling

Look for:

```csharp
catch(Exception)
{
}
```

or throwing generic exceptions.

### 4. Resource management

Check whether:

- Streams are disposed
- DB connections are disposed
- HttpClient is used correctly

### 5. DI

Check for:

```csharp
new SomeService()
```

inside business classes.

### 6. Logging

Check whether sensitive information is logged.

### 7. Security

Check for:

- Hard-coded passwords
- SQL injection
- Missing authorization
- Secrets in configuration/source control

---

# 14. Which lifetime can be used for unique identifiers?

If the question means a service that generates unique IDs, the DI lifetime depends on how the generator maintains state.

If it is completely stateless and uses a reliable unique ID mechanism such as:

```csharp
Guid.NewGuid()
```

it can generally be registered as **Transient** or Singleton because it does not need per-request state.

If the generator maintains state that must be shared application-wide, Singleton may be appropriate, but it must be thread-safe.

### Interview answer

> The lifetime is not determined by the word "unique". It depends on whether the ID generator has state and whether that state should be shared.

---

# 15. What is Kestrel?

## Kestrel — Chat Summary

### 1. What is Kestrel?

**Kestrel is the cross-platform web server used by ASP.NET Core.**

It is responsible for:

- Listening for HTTP/HTTPS requests
- Accepting connections
- Processing HTTP requests
- Passing requests into the ASP.NET Core middleware pipeline
- Sending responses back to clients

Simple flow:

```text
Client
   ↓
Kestrel
   ↓
ASP.NET Core Middleware
   ↓
Controller
   ↓
Business Logic
   ↓
Database
```

---

### 2. Kestrel behind IIS/Nginx

In production, Kestrel is commonly placed behind a reverse proxy:

```text
Client
   ↓
IIS / Nginx
   ↓
Kestrel
   ↓
ASP.NET Core
   ↓
Controller
```

Think of it as:

```text
IIS / Nginx = Front Door / Reverse Proxy
Kestrel     = Web Server for ASP.NET Core
ASP.NET Core = Application Framework / Pipeline
Controller  = Your API endpoint
```

---

### 3. Why is Kestrel between IIS and the API?

Your API controller isn't itself a web server.

For example:

```csharp
[HttpGet("users")]
public IActionResult GetUsers()
{
    return Ok(users);
}
```

Something needs to:

1. Accept the HTTP request
2. Create/process the HTTP context
3. Pass the request through middleware
4. Perform routing
5. Execute the controller
6. Return the HTTP response

Kestrel provides the **web-server part of this hosting model**.

So conceptually:

```text
IIS
 ↓
"I received the HTTP request"
 ↓
Kestrel
 ↓
"I'll host/process the ASP.NET Core application"
 ↓
Middleware
 ↓
Controller
```

---

### 4. Is IIS/Nginx mandatory?

**No.**

Kestrel can directly receive internet/client requests:

```text
Client
   ↓
Kestrel
   ↓
ASP.NET Core API
```

For example:

```bash
dotnet MyApi.dll
```

can start Kestrel and listen on a port.

IIS/Nginx is commonly added for capabilities such as:

- Reverse proxying
- TLS/SSL termination
- Request filtering
- Load balancing
- Authentication/integration
- Centralized web-server configuration
- Connection management

---

### 5. Does Kestrel make the application OS-independent?

**Partly — but phrase it carefully.**

Kestrel itself is **cross-platform**.

It can run on:

```text
Windows
Linux
macOS
Docker/Linux containers
```

But **Kestrel is not what makes .NET itself cross-platform**.

The better understanding is:

```text
.NET Runtime
     +
ASP.NET Core
     ↓
Cross-platform application

Kestrel
     ↓
Cross-platform web server
```

Therefore, the same ASP.NET Core application can run:

```text
Windows
   ↓
Kestrel
   ↓
ASP.NET Core API
```

or:

```text
Linux
   ↓
Kestrel
   ↓
ASP.NET Core API
```

---

## Important Interview Answer

If the interviewer asks:

**"What is Kestrel?"**

Say:

> **Kestrel is the cross-platform web server used by ASP.NET Core. It handles HTTP requests and passes them into the ASP.NET Core request pipeline. It can run directly or behind a reverse proxy such as IIS or Nginx.**

If they ask:

**"Why use Kestrel behind IIS/Nginx?"**

Say:

> **IIS or Nginx can act as the front-facing reverse proxy, while Kestrel hosts and serves the ASP.NET Core application. Kestrel can also run directly without IIS or Nginx.**

### Final mental model

```text
                 Client
                   │
                   ▼
            IIS / Nginx
          "Front Door"
                   │
                   ▼
                Kestrel
             "Web Server"
                   │
                   ▼
             ASP.NET Core
           "Application"
                   │
                   ▼
              Controller
                   │
                   ▼
             Business Logic
                   │
                   ▼
                Database
```

**One sentence to remember:**

> **Kestrel is the cross-platform web server that hosts ASP.NET Core and handles HTTP communication; IIS/Nginx can optionally sit in front of it as a reverse proxy.**

---

# 16. How does Kestrel work behind IIS/Nginx reverse proxy?

Kestrel can run behind a reverse proxy.

```text
Client
   ↓ HTTPS
IIS / Nginx
   ↓ HTTP/HTTPS
Kestrel
   ↓
ASP.NET Core
```

The reverse proxy can handle:

- TLS termination
- Public exposure
- Load balancing
- Static content
- Security filtering

Kestrel handles the ASP.NET Core application.

### Important

When a reverse proxy terminates HTTPS, ASP.NET Core may receive HTTP internally.

Therefore forwarded headers must be configured correctly so the application knows the original scheme was HTTPS.

---

# 17. How does Kestrel handle requests in a multithreaded environment?

Kestrel is designed for concurrent request processing.

Conceptually:

```text
Request 1 ─┐
Request 2 ─┤
Request 3 ─┼→ Kestrel → ASP.NET Core pipeline
Request 4 ─┤
Request 5 ─┘
```

It uses asynchronous I/O and the .NET runtime's thread pool.

### Important point

Kestrel does not normally create one dedicated thread per request.

For I/O operations:

```text
Request
  ↓
Start async I/O
  ↓
Thread can return to pool
  ↓
I/O completes
  ↓
Continuation resumes
```

This allows many concurrent connections without requiring one blocked thread per request.

---

# 18. Role of IoC in Dependency Injection

IoC means **Inversion of Control**.

Normally:

```text
Class → creates dependency
```

With IoC:

```text
DI Container → creates dependency
             ↓
          Class receives it
```

DI is one common way to implement IoC.

### Example

```csharp
public OrderService(IOrderRepository repository)
```

`OrderService` does not decide which repository implementation to create.

The DI container controls that.

---

# 19. Transient lifetime

Transient means:

> A new instance is created every time the service is requested.

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();
```

Example:

```text
Request
 ├── Service A → EmailService #1
 └── Service B → EmailService #2
```

Use it for:

- Stateless lightweight services
- Small helper services
- Services that do not need shared state

Avoid expensive initialization if the service is requested very frequently.

---

# 20. Scenario: service unique for each user

Be careful with the wording.

If "unique for each HTTP request" is intended:

```csharp
AddScoped()
```

If the requirement truly means:

> The same service instance must be maintained for a particular user across multiple HTTP requests

then normal ASP.NET Core `Scoped` is **not enough**, because scoped lifetime is normally per request.

You would need to persist user state externally, such as:

- Database
- Distributed cache
- Session, where appropriate

### Interview clarification

> Scoped means one instance per request, not one instance permanently per user.

---

# 21. How does Kestrel improve performance?

Kestrel is optimized for high-throughput HTTP workloads.

Key reasons include:

- Asynchronous I/O
- Efficient networking
- Thread-pool based concurrency
- Low allocation approaches in many request paths
- Modern HTTP support
- Cross-platform architecture

### Important

Performance is not only Kestrel.

Application performance also depends on:

```text
Kestrel
+ Middleware
+ Application code
+ Database
+ Network
+ External APIs
+ Caching
```

---

# 22. Benefits of Kestrel

### Benefits

1. Cross-platform.
2. High performance.
3. Built into ASP.NET Core.
4. Supports HTTP/1.1, HTTP/2 and newer HTTP capabilities depending on runtime/version.
5. Works well in containers.
6. Lightweight.
7. Works behind reverse proxies.
8. Integrates naturally with ASP.NET Core.

### Typical production architecture

```text
Internet
   ↓
Load Balancer / Reverse Proxy
   ↓
Kestrel
   ↓
ASP.NET Core
```

---

# 23. What are the methods/types of Dependency Injection?

The common injection styles are:

### 1. Constructor Injection — preferred

```csharp
public OrderService(IRepository repository)
{
    _repository = repository;
}
```

### 2. Method Injection

A dependency is passed into a method.

```csharp
public void Process(IEmailService emailService)
{
}
```

### 3. Property Injection

Dependency is assigned through a property.

```csharp
public IEmailService EmailService { get; set; }
```

The built-in ASP.NET Core DI container primarily encourages constructor injection.

---

# 24. Scoped lifetime

Scoped means:

> One service instance per DI scope.

In a normal ASP.NET Core HTTP request:

```text
Request A
  ├── Service → Instance #1
  └── Service → Instance #1

Request B
  ├── Service → Instance #2
  └── Service → Instance #2
```

This makes it useful for:

- DbContext
- Request-specific services
- Unit-of-work style state

---

# 25. Scenario-based Transient lifetime

Question:

> A service is lightweight, stateless and does not need to preserve state between requests. Which lifetime?

Answer:

```csharp
AddTransient()
```

Why?

```text
Each use → fresh instance
```

It avoids unnecessary shared state and is suitable for lightweight stateless services.

---

# 26. Purpose of IServiceCollection

`IServiceCollection` is used to register dependencies.

builder.Services = IServiceCollection = the place where we register dependencies for Dependency Injection.

Example:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
```

It stores service registrations that are later used by the DI container.

Conceptually:

```text
IServiceCollection
       ↓
Service registrations
       ↓
ServiceProvider
       ↓
Dependency resolution
```

---

# 27. ConfigureKestrel body size configuration

Example:

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 50 * 1024 * 1024;
});
```

Calculate:

```text
50 × 1024 × 1024
= 52,428,800 bytes
= 50 MB
```

It limits the maximum HTTP request body size accepted by Kestrel to approximately 50 MB.

### Why configure it?

To:

- Allow required large uploads.
- Prevent unexpectedly huge requests.
- Reduce resource-exhaustion risk.

---

# 28. Two high-value, high-frequency thread-safe services: Singleton or Transient?

What does thread-safe mean?

Thread-safe means multiple threads can use the same object at the same time without causing incorrect or corrupted results.

If both services are:

- Completely thread-safe
- Stateless
- Expensive to create
- Frequently requested

**Singleton can be appropriate**.

Why?

```text
Singleton:
One object
↓
Created once
↓
Reused many times
↓
Less allocation
↓
Less GC pressure
```

Transient:

```text
Every resolution
↓
New object
↓
More allocations
↓
Potentially more GC work
```

### But important

Do not choose Singleton merely because the service is thread-safe.

Also check:

- Does it contain mutable shared state?
- Does it depend on Scoped services?
- Is it safe to share across requests?
- Is initialization expensive?

### Interview answer

> If the service is truly stateless, thread-safe, and safe for application-wide sharing, Singleton can reduce object creation and improve resource utilization. If each operation needs an independent instance or the service has request-specific state, Transient is more appropriate.

---

# 29. Improve this logging code

Given:

```csharp
_log.LogInformation(
    $"For below clientid {clientid} the customer {customerid} is processed successfully.");
```

### Problem

String interpolation happens before the logger receives the message.

Prefer structured logging.

```csharp
_log.LogInformation(
    "For client {ClientId}, customer {CustomerId} was processed successfully.",
    clientId,
    customerId);
```

What is Serilog?

Serilog is a structured logging library for .NET.

It helps your application record information about what is happening while the application runs.

Why use Serilog?

You can write logs to different destinations, called sinks:

Your Application
↓
ILogger
↓
Serilog
↓
Sinks
↓
Console / File / App Insights / etc.

Important: ILogger is the interface/abstraction; Serilog is the logging implementation.

This follows the same principle you've been studying with Dependency Injection:

Depend on abstraction
↓
ILogger
↓
Implementation can be Serilog

### Benefits

- Better structured logs
- Better searching/filtering
- Avoids unnecessary string formatting
- Works better with Application Insights and log aggregation systems

### Also check

Do not log sensitive information such as:

- Passwords
- Tokens
- Full payment information
- Sensitive personal data

---

# 30. Background class inherited by ApiController — what can be improved?

If a controller directly inherits a background class, that is usually a design smell.

Example:

```csharp
public class EmployeeController : BackgroundWorker
{
}
```

A controller should normally focus on HTTP concerns.

### Better approach

Create an interface:

```csharp
public interface IReportService
{
    Task GenerateAsync();
}
```

Implementation:

```csharp
public class ReportService : IReportService
{
    public async Task GenerateAsync()
    {
        // business logic
    }
}
```

Register:

```csharp
builder.Services.AddScoped<IReportService, ReportService>();
```

Inject into controller:

```csharp
public class EmployeeController : ControllerBase
{
    private readonly IReportService _reportService;

    public EmployeeController(IReportService reportService)
    {
        _reportService = reportService;
    }
}
```

### If it is truly background work

Use:

```text
BackgroundService
IHostedService
Queue
Service Bus
```

instead of making the controller inherit from the worker.

---

# 31. Environment-specific settings in .NET Core

Typical configuration:

```text
appsettings.json
appsettings.Development.json
appsettings.Production.json
```

Example:

```json
{
  "ConnectionStrings": {
    "Default": "..."
  }
}
```

Production-specific values can be supplied through:

- Environment variables
- Azure Key Vault
- Secret managers
- Container/Kubernetes secrets
- Cloud configuration services

### Configuration priority

ASP.NET Core combines multiple configuration providers.

Later/higher-priority providers can override earlier values.

---

# 32. Purpose of MaxRequestBodySize

Configuration:

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 50 * 1024 * 1024;
});
```

Means:

```text
Maximum request body ≈ 50 MB
```

This applies to the request body handled by Kestrel.

Example:

```text
Client
  ↓
50 MB upload → allowed
60 MB upload → may be rejected by this limit
```

### Interview point

Other layers may also have request-size limits, for example:

- IIS
- Reverse proxy
- API gateway
- Application middleware

So changing Kestrel alone may not be enough if another layer has a lower limit.

---

# 33. Lightweight, stateless service — which lifetime?

Usually:

```csharp
AddTransient()
```

Why?

- Stateless
- Lightweight
- No shared state required
- New instance per resolution

Example:

```csharp
builder.Services.AddTransient<IPriceCalculator, PriceCalculator>();
```

---

# 34. Same service instance throughout one HTTP request

Use:

```csharp
AddScoped()
```

Example:

```csharp
builder.Services.AddScoped<IRequestContext, RequestContext>();
```

Within one request:

```text
Controller
    ↓
Service A
    ↓
Service B
```

All get the same scoped instance.

Next request gets a new instance.

---

# 35. Kestrel configuration for performance/resource utilization

Useful areas include:

- Request body limits
- Keep-alive settings
- Connection limits
- HTTP protocol settings
- HTTPS configuration
- Timeouts

### Example

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 50 * 1024 * 1024;
});
```

The key is to configure limits based on real application requirements.

Do not simply increase every limit.

### Interview answer

> Kestrel configuration can protect resources by limiting request size and connections, while appropriate timeout and protocol settings can help the application handle high concurrency efficiently.

---

# 36. Kestrel limit syntax

The exact property is important.

For example:

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxRequestBodySize = 1080 * 2;
});
```

This means:

```text
2160 bytes
```

assuming the value is interpreted as bytes.

### Be careful

The property is:

```csharp
options.Limits.MaxRequestBodySize
```

not:

```csharp
option.limit.MaxRequestBodyLimit
```

Interview questions sometimes intentionally contain incorrect syntax.

---

# 37. Object doesn't need to share state — which DI lifetime?

If an object:

- Does not need shared state
- Is lightweight
- Is stateless

Use:

```csharp
Transient
```

Example:

```csharp
services.AddTransient<IFormatter, Formatter>();
```

---

# 38. AddScoped<RequestContext> for sharing request state

Yes.

```csharp
builder.Services.AddScoped<RequestContext>();
```

A scoped object can hold state for one HTTP request.

Example:

```text
Request
  ↓
RequestContext
  ├── UserId
  ├── CorrelationId
  └── Request-specific data
```

Multiple services in that request can access the same instance.

### Important

Do not put sensitive or inappropriate global state in a scoped context.

---

# 39. IHostedService vs BackgroundService

## IHostedService

Interface:

```csharp
IHostedService
```

You implement:

```csharp
StartAsync()
StopAsync()
```

## BackgroundService

`BackgroundService` is a base class that implements `IHostedService` and provides a convenient pattern for long-running work.

Example:

```csharp
public class ReportWorker : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await DoWorkAsync(stoppingToken);

            await Task.Delay(
                TimeSpan.FromMinutes(1),
                stoppingToken);
        }
    }
}
```

### Simple difference

```text
IHostedService
    ↓
You implement lifecycle yourself

BackgroundService
    ↓
Convenient base class
    ↓
Override ExecuteAsync()
```

---

# 40. HTTPS request appears as HTTP behind IIS

Scenario:

```text
Browser → HTTPS → IIS → HTTP → Kestrel
```
The confusion is because **HTTPS and HTTP here refer to two different connections**.

Imagine IIS is the **security gate**.

### 1. Client → IIS

The client connects to:

```text
https://api.company.com
```

So:

```text
Client
   │
   │ HTTPS 🔒
   ▼
 IIS :443
```

IIS receives HTTPS because **IIS is configured with the SSL/TLS certificate** and is the public-facing server.

---

### 2. IIS → Kestrel

After IIS decrypts the HTTPS traffic, it can forward the request internally:

```text
IIS
 │
 │ HTTP
 ▼
Kestrel :5000
```

Why HTTP?

Because the communication is happening **inside the server/network**, and IIS has already handled the encryption.

So the complete flow is:

```text
               Internet
                  │
                  │ HTTPS 🔒
                  ▼
             ┌─────────┐
             │   IIS   │
             │ TLS     │
             │ ends    │
             └────┬────┘
                  │
                  │ HTTP
                  ▼
             ┌─────────┐
             │ Kestrel│
             └────┬────┘
                  │
                  ▼
            ASP.NET Core
```

### Why not keep HTTPS between IIS and Kestrel?

You **can** configure HTTPS internally too.

But often it isn't necessary when:

```text
Client → IIS
```

is the security boundary and:

```text
IIS → Kestrel
```

is trusted internal communication.

It also avoids having to perform another TLS connection/certificate configuration internally.

---

### The important point

**IIS is not "converting HTTPS to HTTP" for the client.**

The client still has:

```text
HTTPS 🔒
```

The application has a **separate connection**:

```text
IIS → Kestrel = HTTP
```

Think of it as two separate roads:

```text
Road 1:
Client ───── HTTPS ─────> IIS

Road 2:
IIS ─────── HTTP ───────> Kestrel
```

That's why your application might see:

```csharp
Request.Scheme == "http"
```

even though the user entered:

```text
https://api.company.com
```

The `X-Forwarded-Proto: https` header tells ASP.NET Core:

> **"Although I am talking to you over HTTP, the original client connection was HTTPS."**

That's the key concept to remember.


### Solution

Configure forwarded headers correctly.

```csharp
using Microsoft.AspNetCore.HttpOverrides;

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor |
        ForwardedHeaders.XForwardedProto;
});
```

Then:

```csharp
app.UseForwardedHeaders();
```

This should be placed appropriately early in the middleware pipeline, before components that depend on the forwarded scheme.

### Why important?

It can affect:

- Redirect URLs
- HTTPS detection
- Authentication callbacks
- Generated links

---

# 41. Production/development passwords and secrets

Do **not** store production passwords directly in:

```text
appsettings.json
```

or source control.

Use:

### Development

- .NET User Secrets
- Local environment variables

### Production

- Azure Key Vault
- Managed identity
- Kubernetes Secrets
- Environment variables
- Cloud secret-management services

Example configuration concept:

```text
Application
   ↓
Configuration
   ↓
Secret provider
   ↓
Database password
```

### Strong interview answer

> I keep secrets outside source-controlled configuration files. In development I can use User Secrets, and in production I prefer a managed secret store such as Azure Key Vault with managed identity.

---

# 42. Multithreaded code optimization

This is a duplicate of Question 2.

Key points:

```text
Avoid shared mutable state
        ↓
Avoid locks where unnecessary
        ↓
Use async I/O
        ↓
Use thread-safe collections
        ↓
Avoid .Result/.Wait()
        ↓
Keep critical sections small
```

---

# 43. Review code: generic Exception for validation

Given:

```csharp
if (string.IsNullOrWhiteSpace(employee.Name))
{
    throw new Exception("employee Name is null");
}
```

### Problems

1. Generic `Exception`.
2. Message says null, but the condition also includes empty/whitespace.
3. Validation logic could be clearer.
4. For an API, invalid client input should normally result in a suitable 4xx response rather than an unhandled generic exception.

### Better

With ASP.NET Core model validation:

```csharp
public class Employee
{
    [Required]
    public string Name { get; set; } = "";
}
```

Or explicit domain validation:

```csharp
if (string.IsNullOrWhiteSpace(employee.Name))
{
    throw new ArgumentException(
        "Employee name is required.",
        nameof(employee.Name));
}
```

For an API, use appropriate validation handling so the client receives a clear `400 Bad Request`.

---

# 44. Middleware order: Authorization before Authentication

Given:

```csharp
app.UseAuthorization();
app.UseAuthentication();
```

Yes, the order is wrong.

Correct:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### Why?

Authentication determines:

> Who is the user?

Authorization determines:

> Is this user allowed to access this resource?

Therefore:

```text
Authentication
      ↓
User identity created
      ↓
Authorization
      ↓
Access decision
```

If authorization runs first, the user identity may not have been established correctly.

---

# 45. Async method where one Task is not awaited

Given:

```csharp
public async Task ProcessDataAsync()
{
    SaveCustomerAsync();
    await SaveOrderAsync();
}
```

`SaveCustomerAsync()` is called but its returned Task is not awaited.

### What can happen?

The method starts the customer operation and immediately continues to:

```csharp
await SaveOrderAsync();
```

The caller does not wait for `SaveCustomerAsync()` through `ProcessDataAsync()`.

Potential problems:

- Customer save may still be running.
- Exceptions from that task may not be observed normally.
- `ProcessDataAsync()` can complete while customer save is still running.
- Ordering is not guaranteed.

### Better

If operations must happen sequentially:

```csharp
await SaveCustomerAsync();
await SaveOrderAsync();
```

If they are independent and should run concurrently:

```csharp
var customerTask = SaveCustomerAsync();
var orderTask = SaveOrderAsync();

await Task.WhenAll(customerTask, orderTask);
```

---

# 46. External API failure — which log level?

If your application calls an external API and the call fails unexpectedly, normally use:

```csharp
LogError()
```

Example:

```csharp
_logger.LogError(
    exception,
    "Failed to call Customer API for customer {CustomerId}",
    customerId);
```

### Why Error?

The application failed to perform an expected operation.

### Levels

```text
Trace
Debug
Information
Warning
Error
Critical
```

Use `Critical` when the failure represents a severe application/system-level problem, not for every external API failure.

---

# 47. What happens when an unhandled exception occurs in a controller?

Example:

```csharp
public IActionResult Get()
{
    throw new Exception("Something failed");
}
```

If not handled:

```text
Controller
   ↓
Exception
   ↓
Exception handling middleware
   ↓
HTTP error response
```

In development, the developer exception page may expose detailed information.

In production, use exception-handling middleware to return a safe response.

Example:

```csharp
app.UseExceptionHandler("/error");
```

### Important

Do not expose:

- Stack traces
- Database details
- Internal implementation details
- Secrets

to clients.

---

# 48. Repository Pattern

Repository Pattern abstracts data access.

Without repository:

```text
Business Service → EF Core/DB directly
```

With repository:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Example:

```csharp
public interface IEmployeeRepository
{
    Task<Employee?> GetAsync(int id);
    Task AddAsync(Employee employee);
}
```

Implementation:

```csharp
public class EmployeeRepository : IEmployeeRepository
{
    private readonly AppDbContext _db;

    public EmployeeRepository(AppDbContext db)
    {
        _db = db;
    }

    public Task<Employee?> GetAsync(int id)
    {
        return _db.Employees.FindAsync(id).AsTask();
    }

    public async Task AddAsync(Employee employee)
    {
        _db.Employees.Add(employee);
        await _db.SaveChangesAsync();
    }
}
```

### Important interview nuance

EF Core's `DbContext` and `DbSet` already provide repository/unit-of-work-like behavior. Adding another repository layer is useful only when it provides meaningful abstraction or domain-specific behavior.

---

# 49. Middleware

Middleware is software that participates in processing an HTTP request and response.

Example:

```text
Request
 ↓
Exception Middleware
 ↓
Authentication
 ↓
Authorization
 ↓
Routing
 ↓
Controller
 ↓
Response
```

Example:

```csharp
app.Use(async (context, next) =>
{
    // Before next middleware

    await next();

    // After next middleware
});
```

### Common middleware

- Exception handling
- Authentication
- Authorization
- CORS
- Logging
- HTTPS redirection
- Static files

---

# 50. REST API

REST is an architectural style for designing web APIs around resources.

Example:

```text
GET    /employees
GET    /employees/10
POST   /employees
PUT    /employees/10
PATCH  /employees/10
DELETE /employees/10
```

### REST principles

- Resource-oriented URLs
- HTTP methods represent operations
- Stateless requests
- Appropriate HTTP status codes
- Representation such as JSON

Example:

```http
GET /employees/10
```

Response:

```json
{
  "id": 10,
  "name": "John"
}
```

---

# 51. Async/Await

`async/await` allows asynchronous operations without blocking a thread while waiting for I/O.

Example:

```csharp
public async Task<Employee?> GetAsync(int id)
{
    return await _db.Employees.FindAsync(id);
}
```

Flow:

```text
Request
 ↓
Database call
 ↓
Thread does not need to block
 ↓
Database completes
 ↓
Continuation resumes
 ↓
Response
```

### Best practice

Avoid:

```csharp
.Result
.Wait()
```

Use:

```csharp
await
```

---

# 52. Additional core interview question: async vs synchronous code

### Synchronous

```csharp
var result = GetData();
```

The current execution waits until the operation completes.

### Asynchronous

```csharp
var result = await GetDataAsync();
```

For I/O operations, the thread can be returned to the pool while waiting.

### Important

Async does not automatically mean:

> A new thread is created.

For I/O-bound operations, async is mainly about **not blocking threads while waiting**.

---

# 53. HTTPS appears as HTTP behind IIS

Same scenario as Question 40.

```text
Client
  ↓ HTTPS
IIS
  ↓ HTTP
Kestrel
```

Use forwarded headers so ASP.NET Core knows the original request scheme.

```csharp
app.UseForwardedHeaders();
```

Ensure IIS/proxy is configured to send the appropriate forwarded headers.

---

# 54. Review validation code

Given:

```csharp
if (string.IsNullOrWhiteSpace(employee.Name))
{
    throw new Exception("employee Name is null");
}
```

Main improvements:

```text
Generic Exception
       ↓
Use appropriate validation mechanism
       ↓
Clear message
       ↓
Return proper API validation response
```

Prefer:

```csharp
[Required]
public string Name { get; set; } = "";
```

or an appropriate specific exception/domain validation approach.

---

# 55. Where to store DB passwords?

### Development

Use:

```text
.NET User Secrets
Environment variables
```

### Production

Prefer:

```text
Azure Key Vault
Managed Identity
Kubernetes/Cloud Secret Store
Environment variables
```

Avoid:

```text
appsettings.json in Git
Hard-coded password
Source code
```

---

# 56. Additional question: Why should secrets not be stored in configuration files?

Even if `appsettings.json` is convenient, committing secrets creates risk.

Problems:

- Git history may retain old passwords.
- Developers may accidentally expose them.
- Logs/build artifacts may contain them.
- Secret rotation becomes difficult.

Better:

```text
Application
    ↓
Configuration abstraction
    ↓
External secret provider
```

The application reads the secret without storing it in source control.

---

# 57. Server receives an incorrect request body — response?

The exact response depends on **why** the body is incorrect.

### Invalid JSON

Example:

```json
{
  "name": "John"
```

This can result in:

```text
400 Bad Request
```

### Valid JSON but validation fails

Example:

```json
{
  "name": ""
}
```

With ASP.NET Core validation, commonly:

```text
400 Bad Request
```

### Unsupported media type

If the client sends an unsupported content type:

```text
415 Unsupported Media Type
```

### Important interview answer

> I first distinguish malformed syntax, validation failure, and unsupported media type. They can result in different 4xx responses.

---

# 58. Authentication vs Authorization

## Authentication

Answers:

> Who are you?

Example:

```text
Username/password
JWT
OAuth/OIDC
Microsoft Entra ID
```

## Authorization

Answers:

> What are you allowed to do?

Example:

```text
Admin → Can delete users
User  → Cannot delete users
```

### Flow

```text
Authenticate
    ↓
Identity established
    ↓
Authorize
    ↓
Access allowed/denied
```

---

# 59. BackgroundService code review

Example concept:

```csharp
public class ReportDBWorker : BackgroundService
{
    private readonly DbContext _db;

    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        return DoWork(_db.Context);
    }
}
```

### Potential issues

1. `ExecuteAsync` should follow the required `Task` signature.
2. Long-running work should honor `CancellationToken`.
3. Do not inject a scoped `DbContext` directly into a long-lived BackgroundService.
4. Create a scope for scoped dependencies.
5. External calls should be awaited.
6. Exceptions should be handled/logged appropriately.
7. Use asynchronous database operations.

### Better pattern

```csharp
public class ReportWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<ReportWorker> _logger;

    public ReportWorker(
        IServiceScopeFactory scopeFactory,
        ILogger<ReportWorker> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                using var scope = _scopeFactory.CreateScope();

                var db = scope.ServiceProvider
                    .GetRequiredService<AppDbContext>();

                await DoWorkAsync(db, stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogError(
                    ex,
                    "Background report processing failed.");
            }

            await Task.Delay(
                TimeSpan.FromMinutes(1),
                stoppingToken);
        }
    }
}
```

### Key interview point

A `BackgroundService` is long-lived. A `DbContext` is normally scoped. Therefore create a scope for each unit of background work.

---

# 60. `.Result` at the end of a GET method

Example:

```csharp
public IActionResult Get()
{
    var data = _service.GetAsync().Result;

    return Ok(data);
}
```

### Problem

`.Result` blocks the thread while waiting for an asynchronous operation.

Potential consequences:

- Thread-pool starvation
- Poor scalability
- Deadlock risk in some synchronization-context scenarios
- Reduced throughput

### Better

```csharp
public async Task<IActionResult> Get()
{
    var data = await _service.GetAsync();

    return Ok(data);
}
```

### Interview answer

> I would make the whole call chain asynchronous instead of blocking with `.Result`.

---

# 61. Middleware order

Incorrect:

```csharp
app.UseAuthorization();
app.UseAuthentication();
```

Correct:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

Reason:

```text
Authentication → establish identity
Authorization  → check permissions
```

Order matters because authorization depends on the authenticated identity.

---

# 62. What happens if you don't await an asynchronous method?

Example:

```csharp
public async Task ProcessDataAsync()
{
    SaveCustomerAsync();
    await SaveOrderAsync();
}
```

`SaveCustomerAsync()` returns a `Task`, but the task is ignored.

Possible result:

```text
Start Customer Save
       ↓
Continue immediately
       ↓
Start/Await Order Save
       ↓
ProcessDataAsync may complete
       ↓
Customer Save may still be running
```

### Correct sequential version

```csharp
await SaveCustomerAsync();
await SaveOrderAsync();
```

### Correct concurrent version

```csharp
var customerTask = SaveCustomerAsync();
var orderTask = SaveOrderAsync();

await Task.WhenAll(customerTask, orderTask);
```

Choose based on whether the operations depend on each other.

---

# 63. Unhandled exception in a controller

Flow:

```text
Controller action
      ↓
Exception thrown
      ↓
No local handler
      ↓
Exception middleware
      ↓
HTTP error response
```

Production applications should have centralized exception handling.

Example:

```csharp
app.UseExceptionHandler("/error");
```

The client should receive a safe response rather than internal details.

---

# 64. BackgroundService ExecuteAsync external API failure — log level

Use `LogError` when an external API call fails and the operation cannot be completed.

```csharp
try
{
    await _client.GetDataAsync(stoppingToken);
}
catch (Exception ex)
{
    _logger.LogError(
        ex,
        "Failed to retrieve report data.");
}
```

Use `Critical` only when the problem is severe enough to represent a major application/system failure.

Also consider:

- Retry
- Exponential backoff
- Timeout
- Circuit breaker
- Cancellation token
- Dead-letter handling where applicable

---

# 65. Value Type vs Reference Type in C#

## Value type

A value type variable directly represents its value.

Examples:

```csharp
int
double
bool
struct
enum
```

Example:

```csharp
int a = 10;
int b = a;

b = 20;
```

Result:

```text
a = 10
b = 20
```

Changing `b` does not change `a`.

---

## Reference type

A reference type variable refers to an object.

Examples:

```csharp
class
string
array
delegate
interface
```

Example:

```csharp
var employee1 = new Employee
{
    Name = "John"
};

var employee2 = employee1;

employee2.Name = "David";
```

Both variables refer to the same object.

Conceptually:

```text
employee1 ──┐
            ↓
         Employee
         Name=David
            ↑
employee2 ──┘
```

---

## Important memory clarification

Do not answer:

> Value types are always stored on the stack and reference types are always stored on the heap.

That is an oversimplification.

A better senior-level answer:

> Value/reference type describes the semantics of how a value is represented and copied. It does not simply determine stack versus heap placement. The actual storage depends on context, such as locals, object fields, boxing, closures and runtime optimizations.

---

# Quick Revision: DI Lifetimes

| Lifetime  | Instance behavior                                 | Typical use                                   |
| --------- | ------------------------------------------------- | --------------------------------------------- |
| Transient | New instance each resolution                      | Lightweight/stateless services                |
| Scoped    | One instance per scope, normally per HTTP request | DbContext/request state                       |
| Singleton | One instance for application lifetime             | Shared thread-safe/stateless services, caches |

### Easy memory trick

```text
Transient = Every time
Scoped    = Every request
Singleton = One application instance
```

---

# Quick Revision: Kestrel

```text
Kestrel
  ↓
ASP.NET Core web server
  ↓
Accepts HTTP requests
  ↓
Runs middleware pipeline
  ↓
Controller
  ↓
Response
```

Behind IIS:

```text
Browser
   ↓ HTTPS
IIS
   ↓
Kestrel
   ↓
ASP.NET Core
```

---

# Quick Revision: Async/Await

### Avoid

```csharp
.Result
.Wait()
```

### Prefer

```csharp
await SomeMethodAsync();
```

### If independent

```csharp
var task1 = Method1Async();
var task2 = Method2Async();

await Task.WhenAll(task1, task2);
```

---

# Quick Revision: Middleware Order

Common simplified flow:

```text
Exception handling
        ↓
Forwarded Headers
        ↓
HTTPS / Routing
        ↓
Authentication
        ↓
Authorization
        ↓
Endpoints
```

Exact middleware order depends on the application's requirements.

---

# Quick Revision: Production Secrets

```text
Development
    ↓
.NET User Secrets / environment variables

Production
    ↓
Azure Key Vault / managed secret store
    ↓
Managed Identity
```

Avoid:

```text
Password in source code
Password in Git
Production secret in appsettings.json
```

---

# Quick Revision: Code Review Checklist

When an interviewer gives you a code snippet, mentally check:

```text
1. Correctness
2. Exception handling
3. Async/await
4. Thread safety
5. DI
6. SOLID
7. Resource disposal
8. Logging
9. Security
10. Performance
11. Validation
12. Maintainability
```

---

# Senior Interview Answer Pattern

When asked a scenario question, use this structure:

### 1. Identify the requirement

> First I would understand whether the object needs shared state, request state, or no state.

### 2. Give the choice

> I would use Scoped.

### 3. Explain why

> Scoped gives one instance within the HTTP request, so multiple services can share the same request-specific state.

### 4. Give an example

```csharp
builder.Services.AddScoped<IRequestContext, RequestContext>();
```

### 5. Mention the important limitation

> It is not one instance per user across multiple requests; it is one instance per DI scope.

This structure makes answers clear and shows senior-level understanding.

---

# High-Priority Topics to Practice

For a senior .NET interview, be especially comfortable explaining these without memorized wording:

1. **DI lifetimes — Transient / Scoped / Singleton**
2. **Async/await and `.Result`**
3. **Kestrel and reverse proxy**
4. **Authentication vs Authorization**
5. **Middleware order**
6. **BackgroundService and scoped dependencies**
7. **REST API design**
8. **POST / PUT / PATCH**
9. **Idempotency**
10. **Exception handling**
11. **Thread safety**
12. **GC generations**
13. **SOLID and testability**
14. **Repository pattern**
15. **Environment configuration and secrets**
16. **Structured logging**
17. **Kestrel request limits**
18. **Value vs reference types**
19. **Equals / GetHashCode**
20. **Code-review/design-smell questions**

---

# One-Line Interview Definitions

| Topic              | Simple definition                                           |
| ------------------ | ----------------------------------------------------------- |
| Singleton          | One shared instance for the application's lifetime          |
| Transient          | New instance every time it is resolved                      |
| Scoped             | One instance per DI scope, normally one HTTP request        |
| IoC                | Control of object creation is moved away from the class     |
| DI                 | Dependencies are supplied to a class                        |
| Kestrel            | ASP.NET Core's cross-platform web server                    |
| Middleware         | Component that processes HTTP requests/responses            |
| REST               | Resource-oriented API architectural style                   |
| Authentication     | Determines who the user is                                  |
| Authorization      | Determines what the user can access                         |
| Async              | Allows non-blocking asynchronous operations                 |
| Repository         | Abstraction around data access                              |
| Idempotency        | Repeating the same operation produces the same final effect |
| GC                 | Automatically reclaims unreachable managed objects          |
| Gen 0              | Short-lived objects                                         |
| Gen 1              | Objects surviving Gen 0 collections                         |
| Gen 2              | Long-lived objects                                          |
| IServiceCollection | Collection used to register DI services                     |
| BackgroundService  | Convenient base class for long-running hosted work          |
| IHostedService     | Interface for hosted application lifecycle services         |
