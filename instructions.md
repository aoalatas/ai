# Wafra API Instructions

## Purpose
Defines project-specific rules for WafraApi.

---

## Global Context

This project extends global NAR AI configuration from:
`.ai/.global/`

---

## Rules

* Follow NAR architecture
* Use CQRS for all write operations
* Use validation for commands

---

## Application Layer Rules

* Commands must use primitive types
* Do NOT use domain objects in commands

---

## API Rules

* Controllers must be thin
* No business logic in controllers

---

## Scope Control Rules

### Implement only what is required for the feature

* Do NOT introduce unnecessary layers
* Avoid over-engineering

### Minimal First

Start with minimal implementation:
* Command
* Handler
* Aggregate

Add more ONLY if needed.

---

## Pattern Usage Rules

When implementing features, AUTOMATICALLY apply the correct pattern:

### Domain Layer

| Concept | Pattern | Example |
|---------|---------|---------|
| Primitive with meaning (Name, Code, Money) | `value-object` | CustomerName, OrderNumber |
| Entity with identity | `aggregate` | Customer, Order |
| Collection of rules | `specification` | ActiveCustomersSpec |

### Application Layer

| Concept | Pattern |
|---------|---------|
| Write action (Create, Update, Delete) | `command` |
| Read action (Get, List) | `query` |
| Orchestration logic | `command-handler` |

### Rules

✅ **MUST automatically use patterns — do NOT ask**

Example: