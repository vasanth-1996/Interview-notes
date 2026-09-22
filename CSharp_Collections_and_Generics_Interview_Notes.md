# C# Collections & Generics — Beginner to Intermediate Interview Guide

> A structured, interview-friendly guide to C# collections, generics, common methods, interfaces, and practical selection of the right collection.

---

## 1. What Is a Collection?

A **collection** is used to store multiple values or objects in one place.

### Simple Analogy

Think of:

- A normal variable → **one box**
- A collection → **a cupboard containing many boxes**

Different collection types organize those boxes differently.

| Collection | Simple analogy | Typical use |
|---|---|---|
| `List<T>` | Shopping list | Ordered, dynamic data |
| `Dictionary<TKey, TValue>` | Phone book | Lookup by key |
| `HashSet<T>` | Guest list | Unique values |
| `Queue<T>` | Waiting line | First-In, First-Out |
| `Stack<T>` | Pile of plates | Last-In, First-Out |
| `LinkedList<T>` | Connected train coaches | Node-based insertion/removal |

### Without a Collection

```csharp
string employee1 = "Anita";
string employee2 = "Rahul";
string employee3 = "John";
```

This becomes difficult when there are hundreds of employees.

### With a Collection

```csharp
List<string> employees = new List<string>
{
    "Anita",
    "Rahul",
    "John"
};

employees.Add("Priya");
employees.Remove("John");

foreach (string employee in employees)
{
    Console.WriteLine(employee);
}
```

Now you can easily add, remove, search, and loop through employees.

---

## 2. Why Are Collections Used?

Collections are used when an application needs to manage groups of data.

Common industry examples:

- Employees returned from a database
- Products displayed on an e-commerce page
- Orders waiting for processing
- User roles and permissions
- Cached records
- API response data
- Background jobs
- Unique IDs that have already been processed

### Interview Point

> Choose the collection based on **how you need to access and manage the data**, not simply because it is available.

---

# 3. What Are Generics?

**Generics** allow you to specify the type of data that a class, method, or collection can work with.

### Simple Analogy

A generic collection is like a container with a label:

```text
List<int>       → stores integers
List<string>    → stores strings
List<Employee>  → stores Employee objects
```

```csharp
List<int> numbers = new List<int>();
List<string> names = new List<string>();
List<Employee> employees = new List<Employee>();
```

The type inside `< >` is called the **type parameter**.

---

## 4. Why Are Generics Important?

Generics provide:

1. **Type safety** — only the correct type can be added.
2. **Less casting** — values are returned in their actual type.
3. **Reusable code** — the same code can work with different types.
4. **Better maintainability** — the intended data type is clear.
5. **Better performance** — avoids unnecessary boxing and unboxing.

### Without Generics

```csharp
ArrayList values = new ArrayList();

values.Add(100);
values.Add("Hello");

// Compiles, but fails at runtime
int number = (int)values[1];
```

The collection allowed both integers and strings, which caused a runtime error.

### With Generics

```csharp
List<int> values = new List<int>();

values.Add(100);

// Compile-time error
// values.Add("Hello");
```

The compiler prevents invalid data before the application runs.

> **Modern .NET recommendation:** Prefer generic collections such as `List<T>` and `Dictionary<TKey, TValue>` over older collections such as `ArrayList` and `Hashtable`.

---

# 5. Main Collection Types

| Collection | Analogy | Best use |
|---|---|---|
| Array | Fixed row of boxes | Fixed-size data |
| `List<T>` | Expandable shopping list | General-purpose ordered data |
| `Dictionary<TKey, TValue>` | Phone book | Lookup using a key |
| `HashSet<T>` | Duplicate-free guest list | Unique values |
| `Queue<T>` | Waiting line | FIFO processing |
| `Stack<T>` | Stack of plates | LIFO processing |
| `LinkedList<T>` | Connected train coaches | Frequent insertion/removal between nodes |

---

# 6. Array

An **array** stores items of the same type and has a **fixed size**.

### Analogy

An array is like a row of fixed seats.

You can change who sits in a seat, but you cannot add more seats after creating the array.

```csharp
int[] marks = { 70, 40, 90, 60 };
```

## Important Array Members

| Member | Purpose |
|---|---|
| `Length` | Returns number of elements |
| `[index]` | Gets or updates an item by position |
| `Array.Sort()` | Sorts the array |
| `Array.Reverse()` | Reverses the array |
| `Array.IndexOf()` | Finds the position of a value |
| `Array.Exists()` | Checks whether a condition is true |

### Add and Access Values

Arrays cannot grow using `Add()`. Values are assigned using an index.

```csharp
int[] marks = new int[3];

marks[0] = 80;
marks[1] = 90;
marks[2] = 70;

Console.WriteLine(marks[0]);     // 80
Console.WriteLine(marks.Length); // 3
```

### Sort and Search

```csharp
int[] marks = { 70, 40, 90, 60 };

Array.Sort(marks);
// 40, 60, 70, 90

int position = Array.IndexOf(marks, 70);
// position = 2

bool hasFailedMark = Array.Exists(
    marks,
    mark => mark < 50
);
// true
```

### When to Use an Array

Use an array when:

- The size is fixed.
- You need simple index-based access.
- You are working with fixed configuration values.
- You are processing numeric or low-level data.

---

# 7. List<T>

`List<T>` is a **dynamic, ordered collection** and is one of the most commonly used collections in .NET applications.

### Analogy

A `List<T>` is like a shopping bag. You can add and remove items whenever required.

```csharp
List<string> employees = new List<string>
{
    "Anita",
    "Rahul",
    "John"
};
```

## Important List<T> Properties

| Property / Member | Purpose |
|---|---|
| `Count` | Number of items currently stored |
| `Capacity` | Internal storage currently allocated |
| `[index]` | Gets or updates an item |

### Count

```csharp
Console.WriteLine(employees.Count); // 3
```

`Count` changes when items are added or removed.

### Capacity

```csharp
Console.WriteLine(employees.Capacity);
```

`Capacity` is the amount of internal storage currently available.

Normally, you do not need to manage it manually. It becomes useful when optimizing very large lists.

### Indexer

```csharp
Console.WriteLine(employees[0]); // Anita

employees[1] = "Priya";

Console.WriteLine(employees[1]); // Priya
```

Indexes start at **0**.

---

## Important List<T> Methods

### Add()

Adds one item to the end of the list.

```csharp
List<string> skills = new List<string>();

skills.Add("C#");
skills.Add("SQL");

Console.WriteLine(skills.Count); // 2
```

**Application:** Use `Add()` when a new product, employee, order, or item is created.

---

### AddRange()

Adds multiple items at once.

```csharp
List<string> skills = new List<string>();

skills.AddRange(new List<string>
{
    "C#",
    "SQL",
    "ASP.NET Core"
});
```

You can also use an array:

```csharp
skills.AddRange(new[]
{
    "Azure",
    "Docker"
});
```

**Application:** Useful for adding database results, API response items, multiple products, or multiple permissions.

---

### Insert()

Adds an item at a specific position.

```csharp
List<string> priorities = new List<string>
{
    "Medium",
    "Low"
};

priorities.Insert(0, "High");
```

Result:

```text
High
Medium
Low
```

**Application:** Use `Insert()` when the position of the new item matters.

---

### Remove()

Removes the **first matching value**.

```csharp
List<string> users = new List<string>
{
    "Anita",
    "Rahul",
    "John",
    "Rahul"
};

users.Remove("Rahul");
```

Only the first matching `"Rahul"` is removed.

`Remove()` returns a `bool`:

```csharp
bool removed = users.Remove("John");

Console.WriteLine(removed); // True
```

If the item does not exist:

```csharp
bool removed = users.Remove("Meera");

Console.WriteLine(removed); // False
```

---

### RemoveAt()

Removes an item using its index.

```csharp
List<string> users = new List<string>
{
    "Anita",
    "Rahul",
    "John"
};

users.RemoveAt(1);
```

Result:

```text
Anita
John
```

> **Warning:** The index must be valid. Otherwise, `ArgumentOutOfRangeException` occurs.

```csharp
// users.RemoveAt(10);
```

---

### RemoveAll()

Removes every item that matches a condition.

```csharp
List<int> numbers = new List<int>
{
    10, 15, 20, 25, 30
};

numbers.RemoveAll(number => number > 20);
```

Remaining values:

```text
10
15
20
```

**Application:**

- Remove inactive users
- Remove cancelled orders
- Remove expired records
- Remove values matching a business rule

```csharp
employees.RemoveAll(employee => !employee.IsActive);
```

---

### Contains()

Checks whether a value exists.

```csharp
List<string> roles = new List<string>
{
    "Admin",
    "Manager",
    "Employee"
};

bool hasAdminRole = roles.Contains("Admin");

Console.WriteLine(hasAdminRole); // True
```

Useful for checking:

- Whether a role exists
- Whether a product is in a cart
- Whether a permission is assigned
- Whether an ID has already been added

---

### IndexOf()

Returns the position of the first matching value.

```csharp
int position = roles.IndexOf("Manager");

Console.WriteLine(position); // 1
```

If the value does not exist, it returns `-1`.

```csharp
int position = roles.IndexOf("Guest");

Console.WriteLine(position); // -1
```

---

### Find()

Returns the **first object** that matches a condition.

```csharp
List<Employee> employees = new List<Employee>
{
    new Employee
    {
        Id = 101,
        Name = "Anita",
        IsActive = true
    },
    new Employee
    {
        Id = 102,
        Name = "Rahul",
        IsActive = false
    }
};

Employee? employee = employees.Find(
    employee => employee.Id == 102);

Console.WriteLine(employee?.Name); // Rahul
```

If no item matches, `Find()` returns `null` for a reference type.

---

### FindAll()

Returns **all objects** that match a condition.

```csharp
List<Employee> activeEmployees = employees.FindAll(
    employee => employee.IsActive);

foreach (Employee employee in activeEmployees)
{
    Console.WriteLine(employee.Name);
}
```

**Application:**

- Active employees
- Products in a category
- Orders with a status
- Users with a specific role

---

### Sort()

Sorts the list in ascending order.

```csharp
List<int> scores = new List<int>
{
    80, 50, 95, 60
};

scores.Sort();
```

Result:

```text
50
60
80
95
```

For objects, provide a comparison rule:

```csharp
employees.Sort(
    (first, second) =>
        first.Name.CompareTo(second.Name));
```

---

### Clear()

Removes all items.

```csharp
List<string> cart = new List<string>
{
    "Laptop",
    "Mouse"
};

cart.Clear();

Console.WriteLine(cart.Count); // 0
```

Useful for:

- Emptying a shopping cart
- Resetting a temporary list
- Clearing a batch after processing
- Removing all selected items

---

### ToArray()

Converts a list into an array.

```csharp
List<int> numbers = new List<int>
{
    10, 20, 30
};

int[] numberArray = numbers.ToArray();
```

Use `ToArray()` when another API, library, or method requires an array.

---

## Practical List<T> Example

```csharp
List<Employee> employees = new List<Employee>();

employees.Add(new Employee
{
    Id = 101,
    Name = "Anita",
    IsActive = true
});

employees.AddRange(new List<Employee>
{
    new Employee
    {
        Id = 102,
        Name = "Rahul",
        IsActive = false
    },
    new Employee
    {
        Id = 103,
        Name = "John",
        IsActive = true
    }
});

Employee? selectedEmployee = employees.Find(
    employee => employee.Id == 102);

List<Employee> activeEmployees = employees.FindAll(
    employee => employee.IsActive);

employees.RemoveAll(
    employee => !employee.IsActive);
```

This demonstrates:

- Adding one item
- Adding multiple items
- Finding one item
- Finding multiple items
- Removing items based on a condition

---

# 8. Dictionary<TKey, TValue>

A dictionary stores data as **key-value pairs**.

### Analogy

Think of a phone book:

```text
Key                  Value
--------------------------------
Employee ID          Employee details
Product ID           Product details
Country Code         Country
Error Code           Error message
```

```csharp
Dictionary<int, string> employees =
    new Dictionary<int, string>
    {
        { 101, "Anita" },
        { 102, "Rahul" },
        { 103, "John" }
    };
```

Here:

- `int` = key type
- `string` = value type
- Each key must be unique

---

## Important Dictionary Members

| Member | Purpose |
|---|---|
| `Count` | Number of key-value pairs |
| `Keys` | Collection of all keys |
| `Values` | Collection of all values |
| `[key]` | Gets or updates a value using a key |

### Access Using a Key

```csharp
string employeeName = employees[101];

Console.WriteLine(employeeName); // Anita
```

### Important Warning

If the key does not exist, direct access throws `KeyNotFoundException`.

```csharp
// string name = employees[999];
```

---

## Add()

Adds a new key-value pair.

```csharp
Dictionary<int, string> products =
    new Dictionary<int, string>();

products.Add(1, "Laptop");
products.Add(2, "Mouse");
```

If the key already exists, `Add()` throws an exception.

```csharp
// products.Add(1, "Keyboard");
```

---

## TryAdd()

Adds a pair only if the key does not already exist.

```csharp
bool added = products.TryAdd(1, "Keyboard");

Console.WriteLine(added); // False
```

> Use `TryAdd()` when duplicate keys are possible and you do not want an exception.

---

## ContainsKey()

Checks whether a key exists.

```csharp
if (products.ContainsKey(2))
{
    Console.WriteLine("Product exists");
}
```

Use it when you want to explicitly check for key existence.

---

## TryGetValue()

Safely gets a value using a key.

```csharp
if (products.TryGetValue(
    2,
    out string? productName))
{
    Console.WriteLine(productName);
}
else
{
    Console.WriteLine("Product not found");
}
```

### Why Is TryGetValue() Useful?

It:

- Avoids `KeyNotFoundException`
- Performs lookup and retrieval together
- Is commonly used in production code

### Interview Tip

Instead of:

```csharp
if (products.ContainsKey(id))
{
    var product = products[id];
}
```

prefer:

```csharp
if (products.TryGetValue(id, out var product))
{
    // use product
}
```

This expresses the intent directly and avoids performing a separate lookup.

---

## Remove()

Removes a key-value pair using its key.

```csharp
bool removed = products.Remove(2);

Console.WriteLine(removed); // True
```

If the key does not exist:

```csharp
bool removed = products.Remove(999);

Console.WriteLine(removed); // False
```

---

## Clear()

```csharp
products.Clear();

Console.WriteLine(products.Count); // 0
```

Removes all key-value pairs.

---

## Updating a Value

You can update a value using the indexer.

```csharp
products[1] = "Gaming Laptop";
```

If key `1` exists, its value is updated.

If key `5` does not exist, this syntax creates a new pair:

```csharp
products[5] = "Keyboard";
```

---

## Reading Keys and Values

### Keys

```csharp
foreach (int productId in products.Keys)
{
    Console.WriteLine(productId);
}
```

### Values

```csharp
foreach (string productName in products.Values)
{
    Console.WriteLine(productName);
}
```

### Key and Value Together

```csharp
foreach (KeyValuePair<int, string> product in products)
{
    Console.WriteLine(
        $"Id: {product.Key}, Name: {product.Value}");
}
```

---

## Practical Dictionary Example

```csharp
Dictionary<int, Employee> employeeCache =
    new Dictionary<int, Employee>();

employeeCache.Add(101, new Employee
{
    Id = 101,
    Name = "Anita",
    IsActive = true
});

if (employeeCache.TryGetValue(
    101,
    out Employee? employee))
{
    Console.WriteLine(employee.Name);
}

employeeCache[101].IsActive = false;

employeeCache.Remove(101);
```

### Industry Uses

Dictionaries are commonly used for:

- Caching users by ID
- Finding products by SKU
- Mapping country codes
- Storing configuration settings
- Mapping error codes to messages
- Counting occurrences

---

# 9. HashSet<T>

A `HashSet<T>` stores **unique values**.

### Analogy

A guest list allows each person to appear only once.

```csharp
HashSet<string> skills = new HashSet<string>();

skills.Add("C#");
skills.Add("SQL");
skills.Add("C#");

Console.WriteLine(skills.Count); // 2
```

The duplicate `"C#"` is ignored.

---

## Important HashSet<T> Members

| Member | Purpose |
|---|---|
| `Count` | Number of unique values |
| `Add()` | Adds a value if it does not exist |
| `Contains()` | Checks whether a value exists |
| `Remove()` | Removes a value |
| `UnionWith()` | Combines two sets |
| `IntersectWith()` | Keeps common values |
| `ExceptWith()` | Removes values found in another set |
| `Clear()` | Removes all values |

---

## Duplicate Check Using Add()

One particularly useful behavior is that `Add()` tells you whether the value was actually added.

```csharp
HashSet<int> processedOrderIds =
    new HashSet<int>();

if (processedOrderIds.Add(5001))
{
    Console.WriteLine("Process order");
}
else
{
    Console.WriteLine("Order already processed");
}
```

`Add()` returns:

- `true` → value was added
- `false` → value already existed

### Industry Use

Use `HashSet<T>` for:

- Unique roles
- Unique permissions
- Processed message IDs
- Unique email addresses
- Duplicate prevention

---

## Set Comparison

```csharp
HashSet<string> userRoles =
    new HashSet<string>
    {
        "Employee",
        "Manager"
    };

HashSet<string> requiredRoles =
    new HashSet<string>
    {
        "Manager",
        "Admin"
    };

userRoles.IntersectWith(requiredRoles);

Console.WriteLine(
    userRoles.Contains("Manager")); // True
```

---

# 10. Queue<T>

A queue follows **FIFO**:

> **First In, First Out**

### Analogy

A queue at a ticket counter: the first person who joins is served first.

```csharp
Queue<string> supportTickets =
    new Queue<string>();

supportTickets.Enqueue("Ticket-101");
supportTickets.Enqueue("Ticket-102");
supportTickets.Enqueue("Ticket-103");
```

## Important Queue Members

| Member | Purpose |
|---|---|
| `Count` | Number of waiting items |
| `Enqueue()` | Adds an item at the end |
| `Dequeue()` | Removes and returns the first item |
| `Peek()` | Reads the first item without removing it |
| `Contains()` | Checks whether an item exists |
| `Clear()` | Removes all items |

### Enqueue(), Peek(), Dequeue()

```csharp
Queue<string> jobs = new Queue<string>();

jobs.Enqueue("Job-101");
jobs.Enqueue("Job-102");

Console.WriteLine(jobs.Peek());
// Job-101; still remains in the queue

string nextJob = jobs.Dequeue();

Console.WriteLine(nextJob);
// Job-101; removed from the queue
```

### Safe Dequeue

`Dequeue()` throws an exception if the queue is empty.

```csharp
if (jobs.Count > 0)
{
    string job = jobs.Dequeue();
    Console.WriteLine(job);
}
```

### Industry Use

Use queues for:

- Background jobs
- Email processing
- Notification processing
- Support tickets
- Print requests
- Order processing

---

# 11. Stack<T>

A stack follows **LIFO**:

> **Last In, First Out**

### Analogy

A stack of plates: the last plate placed on top is removed first.

```csharp
Stack<string> undoActions =
    new Stack<string>();

undoActions.Push("Add product");
undoActions.Push("Update quantity");
undoActions.Push("Apply discount");
```

## Important Stack Members

| Member | Purpose |
|---|---|
| `Count` | Number of items |
| `Push()` | Adds an item to the top |
| `Pop()` | Removes and returns the top item |
| `Peek()` | Reads the top item without removing it |
| `Contains()` | Checks whether an item exists |
| `Clear()` | Removes all items |

### Push(), Peek(), Pop()

```csharp
Console.WriteLine(undoActions.Peek());
// Apply discount

string lastAction = undoActions.Pop();

Console.WriteLine(lastAction);
// Apply discount
```

### Safe Pop

```csharp
if (undoActions.Count > 0)
{
    string action = undoActions.Pop();
    Console.WriteLine(action);
}
```

### Industry Use

Use stacks for:

- Undo and redo
- Browser history
- Backtracking
- Expression evaluation
- Navigation history

---

# 12. LinkedList<T>

A linked list stores data in **connected nodes**.

### Analogy

Think of a train where each coach is connected to the next coach.

```csharp
LinkedList<string> workflow =
    new LinkedList<string>();

workflow.AddLast("Development");
workflow.AddLast("Testing");
```

## Important Members

| Member | Purpose |
|---|---|
| `Count` | Number of nodes |
| `First` | First node |
| `Last` | Last node |
| `AddFirst()` | Adds at the beginning |
| `AddLast()` | Adds at the end |
| `AddBefore()` | Inserts before a node |
| `AddAfter()` | Inserts after a node |
| `Find()` | Finds a node |
| `Remove()` | Removes a node or value |

### Insert Before an Existing Node

```csharp
LinkedListNode<string>? testingNode =
    workflow.Find("Testing");

if (testingNode != null)
{
    workflow.AddBefore(
        testingNode,
        "Code Review");
}
```

The workflow becomes:

```text
Development
Code Review
Testing
```

### When to Use LinkedList<T>

Use it when:

- Frequent insertion/removal in the middle is required.
- You already have a reference to a node.
- Index-based access is not important.

> For most business applications, `List<T>` is simpler and more common.

---

# 13. Generic Interfaces

Interfaces describe what a collection can do.

Using collection interfaces in method parameters makes code more flexible because the caller can provide different implementations.

---

## IEnumerable<T>

Use `IEnumerable<T>` when a method only needs to **read or iterate** through data.

```csharp
public void PrintNames(
    IEnumerable<string> names)
{
    foreach (string name in names)
    {
        Console.WriteLine(name);
    }
}
```

This method can accept:

- `List<string>`
- `string[]`
- `HashSet<string>`
- Other enumerable collections

### Key Point

`IEnumerable<T>` does not guarantee that you can add or remove items.

---

## ICollection<T>

Use `ICollection<T>` when basic collection operations such as add, remove, and count are required.

```csharp
public void AddDepartment(
    ICollection<string> departments)
{
    departments.Add("Finance");
}
```

Common members include:

- `Add()`
- `Remove()`
- `Contains()`
- `Count`
- `Clear()`

---

## IList<T>

Use `IList<T>` when **order and index-based access** are required.

```csharp
public void UpdateCity(
    IList<string> cities)
{
    cities[0] = "Mumbai";
}
```

It supports:

- Index access
- `Add()`
- `Remove()`
- `Insert()`
- `IndexOf()`

---

## IDictionary<TKey, TValue>

Use this interface when data has **keys and values**.

```csharp
public string? FindEmployee(
    IDictionary<int, string> employees,
    int employeeId)
{
    employees.TryGetValue(
        employeeId,
        out string? name);

    return name;
}
```

### Interview Point

Instead of tightly coupling a method to `List<T>` or `Dictionary<TKey, TValue>`, use the smallest interface that provides the behavior the method actually needs.

---

# 14. Generic Classes

A generic class can work with different types.

```csharp
public class Box<T>
{
    public T Value { get; set; } = default!;
}
```

### Usage

```csharp
Box<int> numberBox = new Box<int>
{
    Value = 100
};

Box<string> textBox = new Box<string>
{
    Value = "Hello"
};
```

The same class works for integers, strings, employees, products, and other types.

---

# 15. Generic Methods

A generic method can work with different types.

```csharp
public static void PrintValue<T>(T value)
{
    Console.WriteLine(value);
}
```

### Usage

```csharp
PrintValue(100);
PrintValue("Hello");
PrintValue(true);
```

C# identifies the type automatically in most cases.

---

# 16. Generic Constraints

Constraints restrict which types can be used with a generic class or method.

| Constraint | Meaning |
|---|---|
| `where T : class` | `T` must be a reference type |
| `where T : struct` | `T` must be a value type |
| `where T : new()` | `T` must have a public parameterless constructor |
| `where T : BaseClass` | `T` must inherit from `BaseClass` |
| `where T : IInterface` | `T` must implement the interface |

### `class` Constraint

```csharp
public class Service<T>
    where T : class
{
}
```

### `new()` Constraint

```csharp
public class Factory<T>
    where T : new()
{
    public T Create()
    {
        return new T();
    }
}
```

This works only when `T` has a public parameterless constructor.

---

# 17. Industry Example — E-Commerce Order System

Consider an e-commerce order system.

```csharp
public class Order
{
    public int Id { get; set; }

    public string CustomerName { get; set; } = "";

    public decimal Amount { get; set; }
}
```

Different collections solve different requirements:

```csharp
List<Order> allOrders =
    new List<Order>();

Dictionary<int, Order> ordersById =
    new Dictionary<int, Order>();

Queue<Order> pendingOrders =
    new Queue<Order>();

HashSet<int> processedOrderIds =
    new HashSet<int>();
```

### Add an Order

```csharp
Order order = new Order
{
    Id = 5001,
    CustomerName = "Anita",
    Amount = 2500
};

allOrders.Add(order);

ordersById[order.Id] = order;

pendingOrders.Enqueue(order);
```

### Process the Next Order

```csharp
if (pendingOrders.Count > 0)
{
    Order nextOrder = pendingOrders.Dequeue();

    if (processedOrderIds.Add(nextOrder.Id))
    {
        Console.WriteLine(
            $"Processing order {nextOrder.Id}");
    }
    else
    {
        Console.WriteLine(
            "Order was already processed");
    }
}
```

### Find an Order Quickly

```csharp
if (ordersById.TryGetValue(
    5001,
    out Order? foundOrder))
{
    Console.WriteLine(
        foundOrder.CustomerName);
}
```

### What Each Collection Is Doing

| Collection | Responsibility |
|---|---|
| `List<Order>` | Maintains all orders |
| `Dictionary<int, Order>` | Finds an order quickly by ID |
| `Queue<Order>` | Processes orders in arrival order |
| `HashSet<int>` | Prevents duplicate processing |

### Key Design Lesson

A real application may use **multiple collections for the same business entity**, because each collection solves a different access problem.

---

# 18. Quick Reference — Important Methods

## Array

| Member | Usage |
|---|---|
| `Length` | Get number of elements |
| `[index]` | Read or update an element |
| `Array.Sort()` | Sort values |
| `Array.Reverse()` | Reverse values |
| `Array.IndexOf()` | Find the position of a value |
| `Array.Exists()` | Check whether a condition matches |

## List<T>

| Member | Usage |
|---|---|
| `Count` | Current number of items |
| `Capacity` | Internal allocated storage |
| `[index]` | Read or update an item |
| `Add()` | Add one item |
| `AddRange()` | Add multiple items |
| `Insert()` | Add at a specific position |
| `Remove()` | Remove first matching value |
| `RemoveAt()` | Remove using an index |
| `RemoveAll()` | Remove all matching items |
| `Contains()` | Check whether a value exists |
| `IndexOf()` | Find an item position |
| `Find()` | Find the first matching object |
| `FindAll()` | Find all matching objects |
| `Sort()` | Sort the list |
| `Clear()` | Remove all items |
| `ToArray()` | Convert to an array |

## Dictionary<TKey, TValue>

| Member | Usage |
|---|---|
| `Count` | Count key-value pairs |
| `Keys` | Access all keys |
| `Values` | Access all values |
| `[key]` | Read or update by key |
| `Add()` | Add a new key-value pair |
| `TryAdd()` | Add only if key is not present |
| `ContainsKey()` | Check whether a key exists |
| `TryGetValue()` | Safely retrieve a value |
| `Remove()` | Remove using a key |
| `Clear()` | Remove all pairs |

## HashSet<T>

| Member | Usage |
|---|---|
| `Count` | Count unique values |
| `Add()` | Add a value and detect duplicates |
| `Contains()` | Check membership |
| `Remove()` | Remove a value |
| `UnionWith()` | Combine sets |
| `IntersectWith()` | Find common values |
| `ExceptWith()` | Remove values from another set |
| `Clear()` | Remove all values |

## Queue<T>

| Member | Usage |
|---|---|
| `Count` | Count waiting items |
| `Enqueue()` | Add to the end |
| `Dequeue()` | Remove the first item |
| `Peek()` | View the first item without removing |
| `Contains()` | Check for an item |
| `Clear()` | Remove all items |

## Stack<T>

| Member | Usage |
|---|---|
| `Count` | Count stack items |
| `Push()` | Add to the top |
| `Pop()` | Remove the top item |
| `Peek()` | View the top item without removing |
| `Contains()` | Check for an item |
| `Clear()` | Remove all items |

---

# 19. Choosing the Correct Collection

This is one of the most important interview topics.

| Requirement | Recommended collection |
|---|---|
| Fixed number of items | Array |
| Dynamic ordered data | `List<T>` |
| Lookup by unique key | `Dictionary<TKey, TValue>` |
| Unique values only | `HashSet<T>` |
| First-in, first-out processing | `Queue<T>` |
| Last-in, first-out processing | `Stack<T>` |
| Frequent node-based insertion/removal | `LinkedList<T>` |
| Read-only iteration | `IEnumerable<T>` |
| Thread-safe key-value access | `ConcurrentDictionary<TKey, TValue>` |
| Thread-safe queue processing | `ConcurrentQueue<T>` |

### Quick Decision Flow

```text
Do I need multiple values?
        |
        +-- No --> Normal variable
        |
        +-- Yes
             |
             +-- Fixed size?
             |      |
             |      +-- Yes --> Array
             |
             +-- Need lookup by unique key?
             |      |
             |      +-- Yes --> Dictionary<TKey, TValue>
             |
             +-- Need unique values only?
             |      |
             |      +-- Yes --> HashSet<T>
             |
             +-- Need FIFO processing?
             |      |
             |      +-- Yes --> Queue<T>
             |
             +-- Need LIFO processing?
             |      |
             |      +-- Yes --> Stack<T>
             |
             +-- Otherwise --> List<T>
```

---

# 20. Interview-Focused Comparisons

## Array vs List<T>

| Array | `List<T>` |
|---|---|
| Fixed size | Dynamic size |
| Uses `Length` | Uses `Count` |
| No `Add()` | Has `Add()` |
| Good for fixed-size data | Good for changing collections |
| Simple and lightweight | More flexible |

### Interview Answer

> I use an array when the number of elements is fixed and `List<T>` when the collection needs to grow or shrink dynamically.

---

## List<T> vs Dictionary<TKey, TValue>

Use `List<T>` when you primarily work with an ordered sequence.

Use `Dictionary<TKey, TValue>` when you frequently need to find an item using a unique key.

Example:

```csharp
List<Employee> employees;
```

Good when you want:

```text
All employees
```

Dictionary:

```csharp
Dictionary<int, Employee> employeesById;
```

Good when you want:

```text
Employee with ID 101
```

---

## Dictionary vs HashSet

### Dictionary

Stores:

```text
Key → Value
```

Example:

```csharp
Dictionary<int, Employee>
```

### HashSet

Stores:

```text
Unique Value
```

Example:

```csharp
HashSet<int>
```

### Interview Answer

> I use a dictionary when I need to associate a value with a unique key. I use a HashSet when I only care about unique values and want to prevent duplicates.

---

## Queue vs Stack

### Queue

```text
FIFO
First In → First Out
```

Example:

```text
Job A
Job B
Job C

Process → A, B, C
```

### Stack

```text
LIFO
Last In → First Out
```

Example:

```text
A
B
C ← top

Process → C, B, A
```

---

# 21. Important Interview Questions

### Beginner

1. What is a collection?
2. Why do we use collections?
3. What are generics?
4. Why are generic collections preferred?
5. What is the difference between an array and `List<T>`?
6. What is `Dictionary<TKey, TValue>`?
7. Can a dictionary contain duplicate keys?
8. What is a `HashSet<T>`?
9. What is FIFO?
10. What is LIFO?

### Intermediate

11. Difference between `List<T>` and `Dictionary<TKey, TValue>`?
12. Difference between `Dictionary<TKey, TValue>` and `HashSet<T>`?
13. What happens if you add a duplicate key using `Dictionary.Add()`?
14. How do you safely retrieve a dictionary value?
15. What is the difference between `ContainsKey()` and `TryGetValue()`?
16. How does `HashSet<T>.Add()` help prevent duplicates?
17. What is the difference between `Count` and `Capacity` in `List<T>`?
18. When would you use `Queue<T>` in a real application?
19. When would you use `Stack<T>`?
20. When would you use `LinkedList<T>`?
21. What is `IEnumerable<T>`?
22. Why would you accept `IEnumerable<T>` instead of `List<T>` in a method?
23. What is `ICollection<T>`?
24. What is `IList<T>`?
25. What are generic constraints?

---

# 22. Practical Interview Scenarios

## Scenario 1 — Prevent Duplicate Employee IDs

Requirement:

> You receive employee IDs and should not process the same ID twice.

Use:

```csharp
HashSet<int> processedEmployeeIds =
    new HashSet<int>();

if (processedEmployeeIds.Add(employeeId))
{
    // Process employee
}
else
{
    // Already processed
}
```

---

## Scenario 2 — Find Employee by ID

Requirement:

> You frequently need to find an employee using employee ID.

Use:

```csharp
Dictionary<int, Employee> employeesById;
```

Then:

```csharp
if (employeesById.TryGetValue(
    employeeId,
    out Employee? employee))
{
    // Employee found
}
```

---

## Scenario 3 — Process Jobs in Arrival Order

Requirement:

> Jobs must be processed in the same order they arrived.

Use:

```csharp
Queue<Job> jobs;
```

```csharp
jobs.Enqueue(job);

Job nextJob = jobs.Dequeue();
```

---

## Scenario 4 — Undo Last User Action

Requirement:

> The most recent action must be undone first.

Use:

```csharp
Stack<string> actions;
```

```csharp
actions.Push("Update profile");

string lastAction = actions.Pop();
```

---

# 23. Common Mistakes to Avoid

### Mistake 1 — Using a List for Every Problem

A `List<T>` is common, but it is not always the best choice.

Ask:

> Do I need unique values, key-based lookup, FIFO, or LIFO?

---

### Mistake 2 — Direct Dictionary Access When the Key May Not Exist

Avoid:

```csharp
var employee = employeesById[id];
```

when the key may not exist.

Prefer:

```csharp
if (employeesById.TryGetValue(
    id,
    out var employee))
{
    // use employee
}
```

---

### Mistake 3 — Using List.Contains() for Large-Scale Membership Checks Without Considering the Access Pattern

If the main requirement is:

> "Has this value already been seen?"

A `HashSet<T>` may better express the requirement.

```csharp
HashSet<int> processedIds = new();

if (processedIds.Add(id))
{
    // First time
}
```

---

### Mistake 4 — Forgetting That Dictionary Keys Must Be Unique

This throws:

```csharp
dictionary.Add(1, "A");
dictionary.Add(1, "B");
```

If duplicate keys are possible, consider:

```csharp
dictionary.TryAdd(1, "B");
```

or explicitly decide how an existing value should be updated.

---

# 24. Final Summary

Remember these rules:

- A **collection** stores multiple values or objects.
- **Generics** make collections and classes type-safe and reusable.
- Use an **array** when the size is fixed.
- Use `List<T>` for normal dynamic and ordered data.
- Use `Dictionary<TKey, TValue>` for lookup by a unique key.
- Use `HashSet<T>` when duplicate values are not allowed.
- Use `Queue<T>` when the oldest item must be processed first.
- Use `Stack<T>` when the newest item must be processed first.
- Use `LinkedList<T>` when frequent insertion or removal between nodes is required.
- Use `IEnumerable<T>` when a method only needs to read or iterate.
- Use `TryGetValue()` for safe dictionary lookup.
- Use `Any()` or `Contains()` for existence checks.
- Use `Find()` or `FindAll()` to search a `List<T>`.
- Use `Count` to know the number of items in most collections.
- Use `Length` for arrays.
- Use `Add()` to insert one item.
- Use `AddRange()` to insert several items.
- Use `Remove()` to remove a matching value.
- Use `RemoveAt()` to remove by position.
- Use `RemoveAll()` to remove items matching a condition.
- Use `Clear()` to empty a collection.

---

# 25. One-Minute Interview Cheat Sheet

```text
ARRAY
  Fixed size
  Length
  Index-based access

LIST<T>
  Dynamic ordered collection
  Count
  Add / AddRange / Remove / RemoveAll
  Find / FindAll
  Sort

DICTIONARY<TKey,TValue>
  Key → Value
  Unique keys
  Fast key-based lookup
  TryGetValue / TryAdd

HASHSET<T>
  Unique values
  Duplicate prevention
  Add() returns false for duplicate

QUEUE<T>
  FIFO
  Enqueue → Peek → Dequeue

STACK<T>
  LIFO
  Push → Peek → Pop

LINKEDLIST<T>
  Node-based
  Useful for insertion/removal around known nodes

IEnumerable<T>
  Read / iterate

ICollection<T>
  Basic collection operations

IList<T>
  Index + collection operations

GENERICS
  Type safety
  Reusable code
  Less casting
```

> **Core interview mindset:** Don't just memorize collection names. Explain **what access pattern or business requirement** led you to choose the collection.
