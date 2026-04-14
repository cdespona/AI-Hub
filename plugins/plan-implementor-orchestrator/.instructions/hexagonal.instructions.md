---
description: 'Hexagonal Architecture (Ports & Adapters) rules for safe, minimal, agent-driven code changes'
applyTo: '**'
---

# Hexagonal Architecture Instructions

Use this file to keep changes aligned with Ports & Adapters and avoid layer leakage.

## Core Objective

Enforce the dependency rule:

- Domain depends on nothing outside itself
- Application orchestrates use cases
- Infrastructure implements domain ports
- Interfaces translate external requests/responses

## Layer Contracts

### Domain (`domain/`)

- Contains business model: aggregates, entities, value objects, domain services
- Defines ports (interfaces) for persistence and external systems
- Must not import framework or adapter code
- Must enforce business invariants

Allowed dependencies:
- Language stdlib
- Other domain types

Forbidden:
- Spring/Quarkus annotations
- JPA/JDBC/HTTP clients
- DTOs from interface layer

### Application (`application/`)

- Implements use cases and orchestration
- Coordinates transactions, authorization, and domain operations
- Calls domain ports, not infrastructure classes directly
- Contains no persistence or transport details

Allowed dependencies:
- Domain
- Transaction/security abstractions

Forbidden:
- SQL, HTTP client details, framework-specific persistence entities

### Infrastructure (`infrastructure/`)

- Implements domain ports
- Maps domain <-> persistence/external representations
- Owns framework-specific details and adapters

Allowed dependencies:
- Domain ports/models
- Framework/libraries (JPA, REST clients, messaging, etc.)

Forbidden:
- Business rules that belong in domain

### Interfaces (`interfaces/`)

- Entry points (REST, messaging, CLI)
- Maps external requests into application commands/queries
- Maps application/domain responses into transport DTOs

Allowed dependencies:
- Application
- Transport-specific frameworks

Forbidden:
- Direct repository access
- Domain mutation logic in controllers/handlers

## Agent Editing Rules

Before editing:

1. Identify the change type: domain rule, use case orchestration, adapter detail, or transport mapping.
2. Place code in the correct layer first; do not "make it work" in the wrong layer.
3. Check whether a port already exists before creating new interfaces.

While editing:

- Prefer minimal, surgical changes
- Keep domain APIs stable unless explicitly required
- Add or update mappers instead of leaking adapter DTOs into domain
- If a use case needs external data, depend on a domain port

After editing:

1. Verify no forbidden imports crossed boundaries
2. Verify business invariants remain in domain
3. Verify adapters implement ports, not the other way around
4. Run tests/build checks relevant to the touched modules

## Dependency Direction (Must Hold)

- `interfaces -> application -> domain`
- `infrastructure -> domain`
- `domain -X-> application/interfaces/infrastructure`
- `application -X-> infrastructure concrete classes` (depend on ports only)

## Common Anti-Patterns

- Annotating domain objects with `@Entity`, `@Service`, `@Repository`
- Putting business decisions in controllers or repositories
- Returning persistence entities from domain ports
- Application service calling a concrete adapter directly
- Creating "shared utils" that bypass layer boundaries

## Good Placement Examples

- "Validate order can be confirmed" -> `domain`
- "Execute place order workflow" -> `application`
- "Persist order via JPA" -> `infrastructure`
- "Map HTTP request to command" -> `interfaces`

## Minimal Review Checklist

- [ ] Domain is framework-free
- [ ] Ports are defined in domain
- [ ] Adapters implement domain ports
- [ ] Use cases orchestrate but do not contain core business rules
- [ ] Interface layer only handles transport concerns
- [ ] No dependency rule violations introduced
