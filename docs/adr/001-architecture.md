# ADR-001: Solution Architecture

## Status
Accepted, 2026-11-11

## Context
TicketRush is a ticket sales system. A seat cannot be sold to more than one person. Seats added to the cart are held for a fixed period; if the purchase is not completed within that time, they are removed from the cart and become available again. When more than one seat is in the cart, either all of them can be purchased or none of them can (all-or-nothing). A standing (general admission) option must also be supported, and venues can be divided into sections for pricing.

These rules are critical and complex, so they must live in a single place and be testable without a database.

## Decision
Clean Architecture is used, with vertical slices inside the Application layer. MediatR is not used.

Project references point inward:
- Domain → nothing
- Application → Domain
- Infrastructure → Application
- Api → Application, Infrastructure (the Infrastructure reference exists only for DI registration in `Program.cs`)

Tests are split into `UnitTests` and `IntegrationTests` projects.

## Alternatives
- **Layered:** Rejected. Business logic would depend directly on the database and external services.
- **Vertical Slice only:** Rejected. Rules that are checked in many places (hold expiry, availability) could be duplicated across handlers, and layer boundaries would be enforced only by discipline, not by the compiler.

## Consequences
**Benefits**
- Inward-pointing references and interfaces reduce coupling.
- Shared rules are not duplicated.
- Interfaces are easy to mock in tests.
- Vertical slices in Application keep all files of a feature in one folder.

**Costs**
- Many interfaces to write; slower initial development.
- More projects and files.

**Revisit when**
- Adding shared behavior (validation, logging) to every handler by hand becomes painful → reconsider MediatR.
- Abstraction becomes excessive → reduce layers and move closer to Vertical Slice.
