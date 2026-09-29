# Architecture

## 1. Purpose

This document defines the high-level architecture of the first version of the Property Listings platform.

The architecture describes how the application is structured, how requests move through the system, how responsibilities are separated, and how the system can evolve as additional product capabilities are introduced.

The goal is to maintain a simple architecture appropriate for the current product while creating clear boundaries that can support future growth.

---

# 2. Architectural Style

The system will begin as a **modular monolith**.

A modular monolith is a single deployable application whose internal codebase is divided into well-defined modules.

Conceptually:

```text
┌─────────────────────────────────────┐
│        Property Platform API        │
│                                     │
│   ┌──────────┐    ┌─────────────┐   │
│   │  Agents  │    │  Listings   │   │
│   └──────────┘    └─────────────┘   │
│                                     │
│           Shared Infrastructure     │
└─────────────────────────────────────┘
                  │
                  ↓
        PostgreSQL + PostGIS
```

The application is deployed as a single service.

This avoids the operational complexity of distributed systems while maintaining logical boundaries between major product capabilities.

---

# 3. Why a Modular Monolith

The product does not currently require independently deployable services.

Introducing microservices at this stage would add concerns such as:

* inter-service communication
* distributed transactions
* network failure handling
* service discovery
* additional deployments
* distributed observability
* message infrastructure
* more complex local development

without solving a current product problem.

A modular monolith provides many of the organizational benefits of service boundaries while keeping operational complexity low.

As the product grows, individual modules can be extracted into separate services if clear scaling, ownership, reliability, or deployment requirements justify doing so.

---

# 4. High-Level Request Flow

A typical request flows through the application as follows:

```text
Client
  │
  ▼
Fastify
  │
  ▼
Route / Controller
  │
  ▼
Request Validation
  │
  ▼
Application Service
  │
  ▼
Repository / Query Layer
  │
  ▼
PostgreSQL + PostGIS
```

Responses travel back through the same path.

```text
Database
   ↓
Repository
   ↓
Service
   ↓
Route
   ↓
HTTP Response
```

Each layer has a specific responsibility.

---

# 5. HTTP Layer

The HTTP layer is responsible for communication with external clients.

It includes:

* route registration
* path parameters
* query parameters
* request bodies
* request validation
* HTTP status codes
* response serialization
* HTTP-specific error translation

Example responsibilities include:

```text
POST /listings
GET /listings
GET /listings/:id
PATCH /listings/:id
DELETE /listings/:id
```

The HTTP layer should not contain SQL queries or substantial business workflows.

Its primary responsibility is translating HTTP requests into application operations.

---

# 6. Validation

Incoming API data must be validated before being passed deeper into the application.

Validation includes concerns such as:

* required fields
* expected data types
* allowed enum values
* numeric ranges
* valid geographic coordinates
* malformed identifiers
* query parameter validation

Zod will be used to define and execute these validations.

Validation protects the deeper layers of the application from malformed external input.

Validation rules should be distinguished from deeper domain invariants.

For example:

```text
HTTP validation:
price must be supplied as a number

Domain invariant:
price cannot represent an invalid listing price
```

Some rules may be reinforced at multiple layers when appropriate.

---

# 7. Application Service Layer

Services represent application use cases.

Examples include:

```text
CreateListing
UpdateListing
DeleteListing
GetListing
SearchListings
CreateAgent
```

A service coordinates the work necessary to complete a use case.

It may:

* enforce application-level rules
* coordinate repositories
* verify referenced entities
* manage workflows
* later coordinate external systems
* define transaction boundaries

Services should not contain raw SQL.

Services should also avoid depending directly on HTTP-specific concepts such as status codes or request objects.

---

# 8. Why Keep a Service Layer

In the initial product, some services may be extremely small.

For example:

```text
getListing(id)
    ↓
listingRepository.findById(id)
```

This is acceptable.

The service layer exists because application workflows are expected to become more complex over time.

For example, creating a listing could later involve:

```text
Create Listing
      │
      ├── verify property
      ├── verify agent permissions
      ├── persist listing
      ├── update search index
      ├── record audit event
      └── trigger notifications
```

These workflows belong in an application service rather than inside HTTP routes or persistence code.

The architecture should therefore allow the service layer to remain simple today and grow naturally when product requirements become richer.

---

# 9. Repository Layer

Repositories isolate persistence concerns from application logic.

The repository layer is responsible for:

* executing SQL
* retrieving database records
* persisting entities
* updating records
* deleting records
* executing search queries
* mapping database results into application representations

Example interface:

```text
ListingRepository

create(...)
findById(...)
update(...)
delete(...)
search(...)
```

The repository should not contain HTTP logic.

It should also avoid implementing unrelated business workflows.

---

# 10. SQL Strategy

The system will use explicit SQL through a thin database access layer rather than a heavy ORM.

This provides direct visibility into:

* SQL queries
* joins
* indexes
* query plans
* constraints
* PostGIS functions
* performance characteristics

This is particularly important because geographic search and query optimization are core capabilities of the platform.

The repository layer provides enough abstraction to prevent raw SQL from leaking throughout the application while preserving direct control over database behavior.

---

# 11. Database

PostgreSQL is the primary source of truth for the system.

PostGIS extends PostgreSQL with geospatial capabilities.

The database is responsible for:

* persistent storage
* relational integrity
* constraints
* transactional consistency
* indexed queries
* geospatial operations

The database should enforce important structural invariants where appropriate rather than relying exclusively on application code.

Exact schema and indexing decisions are defined in the data-model stage.

---

# 12. Module-Oriented Project Structure

The codebase will primarily be organized around product domains rather than technical layers.

Proposed structure:

```text
src/
├── app.ts
├── server.ts
│
├── modules/
│   ├── agents/
│   │   ├── agent.routes.ts
│   │   ├── agent.schemas.ts
│   │   ├── agent.service.ts
│   │   ├── agent.repository.ts
│   │   └── agent.types.ts
│   │
│   └── listings/
│       ├── listing.routes.ts
│       ├── listing.schemas.ts
│       ├── listing.service.ts
│       ├── listing.repository.ts
│       └── listing.types.ts
│
├── infrastructure/
│   └── database/
│       ├── connection.ts
│       └── transaction.ts
│
├── shared/
│   ├── errors/
│   ├── config/
│   └── types/
│
└── plugins/
```

This structure keeps code related to the same domain close together.

---

# 13. Why Domain-Oriented Organization

A purely technical directory structure might look like:

```text
routes/
services/
repositories/
schemas/
```

This works for very small applications.

However, as the system grows, each directory accumulates files from unrelated product areas.

For example:

```text
services/
  listing.service.ts
  booking.service.ts
  payment.service.ts
  property.service.ts
  guest.service.ts
  maintenance.service.ts
  notification.service.ts
```

A module-oriented structure keeps each capability together:

```text
modules/
  listings/
  bookings/
  payments/
  properties/
  guests/
  maintenance/
```

This makes module ownership and future extraction clearer.

---

# 14. Dependency Direction

Dependencies should generally flow inward toward application and domain concepts.

For example:

```text
Route
  ↓
Service
  ↓
Repository
  ↓
Database
```

The repository should never depend on the HTTP route.

The service should not depend on Fastify request objects.

This prevents infrastructure details from spreading unnecessarily through the codebase.

---

# 15. Error Flow

Errors should originate from the layer that understands the problem and be translated appropriately at system boundaries.

Example:

```text
Repository
   ↓
listing not found
   ↓
Service
   ↓
application-level NotFound error
   ↓
HTTP layer
   ↓
404 response
```

A database error should not normally be exposed directly to API consumers.

The HTTP layer converts internal application errors into stable API responses.

The exact error contract will be defined during API design.

---

# 16. Dependency Injection

Dependencies should be constructed explicitly.

For example:

```text
Database
   ↓
ListingRepository
   ↓
ListingService
   ↓
ListingRoutes
```

This enables components to be tested independently.

The project does not require a dependency-injection framework.

Simple constructor or function-based dependency injection is sufficient.

---

# 17. Transaction Boundaries

A transaction should represent one atomic application operation when multiple database changes must succeed or fail together.

For example, a future workflow may require:

```text
BEGIN

Create reservation
Block availability
Create payment record

COMMIT
```

If any operation fails:

```text
ROLLBACK
```

Most initial listing operations may only require a single database statement and therefore may not need explicit transaction orchestration.

Transaction boundaries should be introduced where atomicity requirements exist rather than automatically wrapping every request.

---

# 18. External Integrations

External services should not be called directly from arbitrary application code.

Future integrations might include:

* payment providers
* email providers
* SMS providers
* object storage
* search engines
* mapping services
* identity providers

These integrations should be isolated behind adapters or dedicated infrastructure modules.

Conceptually:

```text
Application Service
       │
       ▼
Interface / Adapter
       │
       ▼
External Provider
```

This prevents external provider APIs from becoming tightly coupled to the domain.

---

# 19. Search Architecture

For the initial version, PostgreSQL remains responsible for structured listing search.

This includes:

* property type filters
* price filters
* bedroom filters
* geographic filtering

PostGIS handles geospatial operations.

A dedicated search engine may later be introduced for capabilities such as:

* full-text ranking
* fuzzy search
* typo tolerance
* autocomplete
* faceted discovery
* relevance scoring

If this occurs, PostgreSQL should remain the system of record while the search engine acts as a derived search representation.

---

# 20. Future Search Evolution

Possible future architecture:

```text
                    ┌─────────────┐
                    │ PostgreSQL  │
                    │   Source    │
                    │  of Truth   │
                    └──────┬──────┘
                           │
                    change/event
                           │
                           ▼
                    ┌─────────────┐
                    │   Search    │
                    │    Index    │
                    └──────┬──────┘
                           │
                           ▼
                       Discovery
```

This avoids treating a search engine as the authoritative transactional database.

---

# 21. Caching

Caching is not required initially.

A cache such as Redis should only be introduced when measurements demonstrate a useful caching opportunity.

Potential future use cases include:

* expensive frequently repeated searches
* sessions
* distributed locks
* rate limiting
* short-lived computed results

Introducing caching without a demonstrated need would add invalidation complexity.

---

# 22. Asynchronous Processing

The initial architecture is primarily synchronous.

A request is received, processed, and returned directly.

Future operations may not belong in the request lifecycle.

Examples include:

* sending emails
* indexing search documents
* processing uploaded images
* generating reports
* notifications
* analytics events

These capabilities may later move to background workers or event-driven workflows.

The current modular boundaries should make this evolution possible without requiring background infrastructure today.

---

# 23. Observability

The application should expose enough information to understand production behavior.

Initial observability includes:

* structured application logs
* request identifiers where useful
* error logging
* health checks

Future observability may include:

* metrics
* distributed tracing
* query performance monitoring
* centralized log aggregation
* alerting

Observability should evolve alongside operational complexity.

---

# 24. Scalability Model

The API should remain stateless wherever practical.

This allows multiple API instances to run behind a load balancer:

```text
                Load Balancer
               /      |      \
              ↓       ↓       ↓
            API 1   API 2   API 3
               \      |      /
                \     |     /
                  PostgreSQL
```

State belongs primarily in external systems such as the database rather than application process memory.

This allows horizontal scaling of the API layer when traffic grows.

---

# 25. Architecture Boundaries

The initial architecture deliberately avoids introducing:

* microservices
* Kubernetes
* Kafka
* distributed transactions
* Redis as a mandatory dependency
* Elasticsearch as a mandatory dependency
* complex event-driven infrastructure
* service meshes

These technologies may become appropriate when a concrete system requirement justifies them.

They are not architectural goals by themselves.

---

# 26. Future Product Modules

As the product expands, the modular monolith could evolve toward:

```text
modules/
├── agents/
├── properties/
├── listings/
├── units/
├── guests/
├── bookings/
├── availability/
├── payments/
├── maintenance/
├── utilities/
├── notifications/
└── search/
```

Each module should represent a coherent product capability.

Some may eventually become independent services if operational requirements justify the separation.

---

# 27. Architectural Principles

The architecture follows several principles.

### Keep boundaries clear

HTTP, application logic, and persistence should not become tightly coupled.

### Prefer explicit behavior

Database queries, dependencies, and workflows should remain understandable.

### Keep infrastructure proportional

Infrastructure should solve existing problems rather than anticipated ones.

### Keep modules cohesive

Code belonging to the same domain capability should remain close together.

### Optimize for change

The architecture should make future changes possible without building those future requirements prematurely.

---

# 28. Initial Architecture

The resulting initial architecture is:

```text
                         Client
                           │
                           ▼
                    ┌─────────────┐
                    │   Fastify   │
                    │     API     │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Validation  │
                    │    Zod      │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  Services   │
                    │ Use Cases   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │Repositories │
                    │  Raw SQL    │
                    └──────┬──────┘
                           │
                           ▼
                ┌────────────────────┐
                │ PostgreSQL         │
                │ + PostGIS          │
                └────────────────────┘
```

This architecture provides a simple foundation for the first version while preserving clear paths toward a larger property platform.
