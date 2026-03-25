# NAR Global Instructions

## Core Principles

* Clean Architecture is mandatory
* Domain layer must be independent
* Application uses CQRS (Command/Handler)
* Controllers must be thin

---

## Layer Rules

### Domain

* Contains business logic only
* No external dependencies

### Application

* Orchestrates use cases
* No direct DB access

### Infrastructure

* Handles persistence only

### API

* Calls Application layer only

---

## Non-Overridable Rules

* No business logic in controllers
* Domain must not depend on infrastructure
