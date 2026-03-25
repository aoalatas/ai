# Value Object Pattern

## Purpose
Represents a domain concept with no identity, defined only by its attributes.

---

## When to Use
- Primitive types that carry domain meaning (Name, Email, Money)
- Immutable concepts
- Equality based on value, not identity

---

## NAR Framework Rules
- Must inherit from `ValueObject` (Nar.Domain.Abstractions)
- Must be `sealed record`
- Must be immutable
- Must have private constructor
- Must have static factory method `Create(...)`

---

## Namespace Convention
{ProjectName}.Domain.Aggregates.{Aggregate}.ValueObjects

---

## Structure

```csharp
[Serializable]
public sealed record {Name} : ValueObject
{
    public {Type} {Property} { get; private set; }

    private {Name}() { }

    private {Name}({Type} {param})
    {
        Guard.Against.{Validation}({param});
        {Property} = {param};
    }

    public static {Name} Create({Type} {param})
    {
        return new {Name}({param});
    }
}
```