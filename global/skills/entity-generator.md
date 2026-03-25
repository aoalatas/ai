# Entity Generator Skill

Sen bir **Senior .NET Developer**sın ve **NarCore Framework** kullanarak enterprise-grade kod yazıyorsun.

---

## Görev

**{entityName}** entity'si için complete bir domain entity oluştur.

### Properties (Özellikler)

{properties}

---

## Framework Context

Aşağıdaki framework kurallarını oku ve **MUTLAKA** uygula:

**Resource:** `resource://context/nar-framework`

---

## Entity Structure - ZORUNLU

### 1. Namespace

```csharp
namespace {ProjectName}.Domain.Aggregates.{AggregateFolder}.Entities
```

Örnek:
- `BankingDemo.Customer.Domain.Aggregates.Bank.Entities`
- `YourProject.Domain.Aggregates.Order.Entities`

### 2. Imports (Using Statements)

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using AlbarakaTech.Nar.Domain.Abstractions;
using AlbarakaTech.Nar.Domain.Abstractions.Entities;
```

Domain event kullanılacaksa:
```csharp
using AlbarakaTech.Nar.EventBus.Domain;
```

### 3. Class Declaration

**Aggregate Root ise:**
```csharp
public class {entityName} : Entity<{KeyType}>, IAggregateRoot
```

**Normal entity ise:**
```csharp
public class {entityName} : Entity<{KeyType}>
```

**Domain event'li aggregate root:**
```csharp
public class {entityName} : EventfulEntity<{KeyType}>, IAggregateRoot
```

---

## Template - Tam Yapı

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using AlbarakaTech.Nar.Domain.Abstractions;
using AlbarakaTech.Nar.Domain.Abstractions.Entities;

namespace {ProjectName}.Domain.Aggregates.{AggregateFolder}.Entities
{
    /// <summary>
    /// {entityName} entity
    /// {Description}
    /// </summary>
    public class {entityName} : Entity<{KeyType}>, IAggregateRoot
    {
        // ============================================
        // PROPERTIES
        // ============================================
        
        /// <summary>
        /// {Property description}
        /// </summary>
        public virtual string PropertyName { get; private set; }
        
        // Navigation properties (eğer varsa)
        private readonly ICollection<ChildEntity> _childItems = new List<ChildEntity>();
        public IReadOnlyCollection<ChildEntity> ChildItems => _childItems.ToList().AsReadOnly();
        
        // ============================================
        // CONSTRUCTORS
        // ============================================
        
        /// <summary>
        /// Protected parameterless constructor for EF Core
        /// </summary>
        protected {entityName}() { }
        
        /// <summary>
        /// Private business constructor
        /// </summary>
        private {entityName}(/* parameters */)
        {
            // Property assignments
            PropertyName = propertyValue;
        }
        
        // ============================================
        // FACTORY METHODS
        // ============================================
        
        /// <summary>
        /// Creates a new {entityName}
        /// </summary>
        public static {entityName} Create(/* parameters */)
        {
            // Business rule validation (eğer varsa)
            // CheckRule(new SomeBusinessRule());
            
            return new {entityName}(/* parameters */);
        }
        
        // ============================================
        // DOMAIN METHODS
        // ============================================
        
        /// <summary>
        /// Updates {entityName} properties
        /// </summary>
        public void Update(/* parameters */)
        {
            PropertyName = newValue;
        }
        
        /// <summary>
        /// Deletes (soft delete) {entityName}
        /// </summary>
        public void Delete()
        {
            IsActive = false;
        }
        
        // Collection operations (eğer child entity varsa)
        public void AddChildItem(/* parameters */)
        {
            var item = ChildEntity.Create(/* parameters */);
            _childItems.Add(item);
        }
    }
}
```

---

## ZORUNLU KURALLAR

### ✅ Rule 1: Base Class

**Aggregate root:**
```csharp
public class Order : Entity<Guid>, IAggregateRoot
```

**Normal entity:**
```csharp
public class OrderItem : Entity<long>
```

**Domain event'li:**
```csharp
public class Customer : EventfulEntity<Guid>, IAggregateRoot
```

**Key type seçimi:**
- `int` - Auto-increment ID için
- `long` - Büyük veri setleri için
- `Guid` - Distributed systems, event sourcing için

---

### ✅ Rule 2: Properties - VIRTUAL

**ZORUNLU:** Tüm properties `virtual` olmalı!

```csharp
public virtual string Name { get; private set; }         // ✅ Doğru
public virtual int Age { get; private set; }             // ✅ Doğru
public virtual bool IsActive { get; set; }               // ✅ Doğru

public string Email { get; private set; }                // ❌ Yanlış - virtual yok
```

**Neden virtual?**
- EF Core lazy loading için gerekli
- Proxy oluşturma için gerekli

---

### ✅ Rule 3: Private Setters

**Business logic koruması için:**

```csharp
public virtual string Name { get; private set; }         // ✅ Sadece domain method'larla değişir
public virtual decimal Amount { get; private set; }      // ✅ Sadece domain method'larla değişir

public virtual string Email { get; set; }                // ❌ Public setter - herkes değiştirebilir
```

---

### ✅ Rule 4: Collections Pattern

**Backing field + ReadOnly:**

```csharp
// Private backing field
private readonly ICollection<OrderItem> _items = new List<OrderItem>();

// Public readonly collection
public IReadOnlyCollection<OrderItem> Items => _items.ToList().AsReadOnly();

// Domain method ile ekleme
public void AddItem(Product product, int quantity)
{
    var item = OrderItem.Create(product, quantity);
    _items.Add(item);
}
```

**Modern C# 12 syntax:**
```csharp
private readonly ICollection<OrderItem> _items = [];
```

---

### ✅ Rule 5: Constructor Pattern

**3 constructor zorunlu:**

```csharp
// 1. Protected parameterless - EF Core için
protected User() { }

// 2. Private business - Domain logic için
private User(string name, string email)
{
    Name = name;
    Email = email;
}

// 3. Static factory - Entity oluşturma için
public static User Create(string name, string email)
{
    return new User(name, email);
}
```

---

### ✅ Rule 6: Static Factory Method

**ZORUNLU:** Entity oluşturma sadece `Create()` method ile:

```csharp
public static {entityName} Create(/* parameters */)
{
    // Business rule validation
    CheckRule(new SomeBusinessRule());
    
    return new {entityName}(/* parameters */);
}
```

**Örnek:**
```csharp
public static User Create(string name, string email, string phone)
{
    CheckRule(new UserEmailMustBeUnique(email));
    
    return new User(name, email, phone);
}
```

---

### ✅ Rule 7: Business Rules (Optional)

**Eğer validation gerekiyorsa:**

```csharp
using AlbarakaTech.Nar.Domain.Abstractions.Rules;

protected static void CheckRule(IBusinessRule rule)
{
    if (!rule.IsValid())
    {
        throw new NarBusinessException(rule.Message, parameters: rule.MessageParameters);
    }
}
```

**Kullanım:**
```csharp
public static Order Create(decimal amount)
{
    CheckRule(new OrderAmountMustBePositive(amount));
    
    return new Order(amount);
}
```

---

### ✅ Rule 8: Domain Events (Optional)

**Eğer domain event gerekiyorsa:**

```csharp
public class Customer : EventfulEntity<Guid>, IAggregateRoot
{
    private Customer(string name)
    {
        base.Id = Guid.NewGuid();
        Name = name;
        
        // Domain event ekle
        base.AddDomainEvent(new CustomerRegisteredEvent(this.Id));
    }
}
```

---

### ✅ Rule 9: XML Documentation

**Her public member için:**

```csharp
/// <summary>
/// Creates a new User with the specified details
/// </summary>
/// <param name="name">User's full name</param>
/// <param name="email">User's email address</param>
/// <returns>A new User instance</returns>
public static User Create(string name, string email)
{
    return new User(name, email);
}
```

---

### ✅ Rule 10: Naming Conventions

**Classes:** PascalCase
- ✅ `User`, `Order`, `OrderItem`
- ❌ `user`, `order_item`

**Properties:** PascalCase
- ✅ `Name`, `Email`, `TotalAmount`
- ❌ `name`, `email`, `total_amount`

**Private fields:** _lowercase
- ✅ `_items`, `_branches`
- ❌ `items`, `Branches`, `m_items`

**Booleans:** Is/Has prefix
- ✅ `IsActive`, `IsDeleted`, `HasPermission`
- ❌ `Active`, `Deleted`, `Permission`

**Methods:** PascalCase + verb
- ✅ `Create()`, `Update()`, `Delete()`, `AddItem()`
- ❌ `create()`, `add_item()`

---

## Property Type Mapping

User'ın verdiği property string'inden C# type'larına çeviri:

### Primitive Types
- `name:string` → `public virtual string Name { get; private set; }`
- `age:int` → `public virtual int Age { get; private set; }`
- `price:decimal` → `public virtual decimal Price { get; private set; }`
- `isActive:bool` → `public virtual bool IsActive { get; private set; }`
- `createdAt:DateTime` → `public virtual DateTime CreatedAt { get; private set; }`

### Nullable Types
- `age:int?` → `public virtual int? Age { get; private set; }`
- `deletedAt:DateTime?` → `public virtual DateTime? DeletedAt { get; private set; }`

### Enums
- `status:OrderStatus` → `public virtual OrderStatus Status { get; private set; }`

### Value Objects
- `address:Address` → `public virtual Address Address { get; private set; }`

### Collections (one-to-many)
- `items:OrderItem[]` → 
  ```csharp
  private readonly ICollection<OrderItem> _items = new List<OrderItem>();
  public IReadOnlyCollection<OrderItem> Items => _items.ToList().AsReadOnly();
  ```

---

## Example Input → Output

### Input
```
entityName: User
properties: Name:string, Email:string, Age:int, IsActive:bool
```

### Output

```csharp
using System;
using AlbarakaTech.Nar.Domain.Abstractions;
using AlbarakaTech.Nar.Domain.Abstractions.Entities;

namespace YourProject.Domain.Aggregates.User.Entities
{
    /// <summary>
    /// User entity
    /// Represents a system user with authentication and profile information
    /// </summary>
    public class User : Entity<long>, IAggregateRoot
    {
        // ============================================
        // PROPERTIES
        // ============================================
        
        /// <summary>
        /// User's full name
        /// </summary>
        public virtual string Name { get; private set; }
        
        /// <summary>
        /// User's email address (unique)
        /// </summary>
        public virtual string Email { get; private set; }
        
        /// <summary>
        /// User's age
        /// </summary>
        public virtual int Age { get; private set; }
        
        /// <summary>
        /// Indicates whether the user is active
        /// </summary>
        public virtual bool IsActive { get; private set; }
        
        // ============================================
        // CONSTRUCTORS
        // ============================================
        
        /// <summary>
        /// Protected parameterless constructor for EF Core
        /// </summary>
        protected User() { }
        
        /// <summary>
        /// Private business constructor
        /// </summary>
        private User(string name, string email, int age)
        {
            Name = name;
            Email = email;
            Age = age;
            IsActive = true;
        }
        
        // ============================================
        // FACTORY METHODS
        // ============================================
        
        /// <summary>
        /// Creates a new User
        /// </summary>
        /// <param name="name">User's full name</param>
        /// <param name="email">User's email address</param>
        /// <param name="age">User's age</param>
        /// <returns>A new User instance</returns>
        public static User Create(string name, string email, int age)
        {
            return new User(name, email, age);
        }
        
        // ============================================
        // DOMAIN METHODS
        // ============================================
        
        /// <summary>
        /// Updates user profile information
        /// </summary>
        public void Update(string name, int age)
        {
            Name = name;
            Age = age;
        }
        
        /// <summary>
        /// Changes user's email address
        /// </summary>
        public void ChangeEmail(string email)
        {
            Email = email;
        }
        
        /// <summary>
        /// Deactivates the user (soft delete)
        /// </summary>
        public void Deactivate()
        {
            IsActive = false;
        }
        
        /// <summary>
        /// Activates the user
        /// </summary>
        public void Activate()
        {
            IsActive = true;
        }
    }
}
```

---

## Checklist - Her Entity İçin Kontrol Et

- [ ] Base class doğru (`Entity<T>` veya `EventfulEntity<T>`)
- [ ] `IAggregateRoot` interface var mı (aggregate root ise)
- [ ] Tüm properties `virtual` mı
- [ ] Business logic için `private set` var mı
- [ ] Protected parameterless constructor var mı
- [ ] Private business constructor var mı
- [ ] `Create()` static factory method var mı
- [ ] XML documentation eklenmiş mi
- [ ] Naming conventions uygun mu
- [ ] Collection pattern doğru mu (backing field + readonly)
- [ ] Domain methods var mı (Update, Delete, vb.)

---

## ÖNEMLİ NOTLAR

1. **Her property virtual olmalı** - EF Core için zorunlu
2. **Private setters** - Business logic koruması
3. **Static factory Create()** - Entity oluşturmanın tek yolu
4. **Protected parameterless constructor** - EF Core için zorunlu
5. **Collections readonly** - Encapsulation
6. **XML documentation** - Her public member için
7. **Business rules** - Domain katmanında validation
8. **Domain events** - State değişikliklerini bildir

---

Şimdi yukarıdaki tüm kurallara göre **{entityName}** entity'sini oluştur!

**MUTLAKA:**
- Tüm properties virtual olsun
- Private setters kullan
- Protected parameterless constructor ekle
- Static Create() factory method ekle
- XML documentation ekle
- Domain methods ekle (Update, Delete)
