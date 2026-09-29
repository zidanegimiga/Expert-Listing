# Property Listings API - Requirements

## 1. Purpose

This document translates the problem definition into concrete system requirements.

The requirements define what the system must do and the quality characteristics it must satisfy.

Implementation decisions such as database schema, API structure, indexing strategy, Docker architecture, and deployment strategy will be defined in subsequent design stages.

---

# 2. Functional Requirements

## FR-01 - Create an Agent

The system must allow an agent record to be created so that listings can reference a valid agent.

An agent should have:

* unique identifier
* name
* email
* optional phone number
* creation timestamp

The system must reject an agent with an invalid or duplicate email where email uniqueness is enforced.

### Acceptance Criteria

* A valid agent can be created.
* The response contains the created agent.
* The agent receives a unique identifier.
* An agent with a duplicate unique email cannot be created.
* Invalid input returns a client error.

---

## FR-02 - Retrieve an Agent

The system must allow an agent to be retrieved by identifier.

### Acceptance Criteria

* Existing agent returns successfully.
* Nonexistent agent returns a not-found response.
* Invalid identifier format returns a validation error where applicable.

---

# 3. Listing Management

## FR-03 - Create a Listing

The system must allow a listing to be created with:

* title
* price
* property type
* number of bedrooms
* geographic location
* agent identifier

The listing may also contain additional descriptive information such as:

* description
* bathrooms
* address
* city
* state
* currency

The system must verify that the referenced agent exists.

### Acceptance Criteria

* A valid listing can be created.
* The listing receives a unique identifier.
* The listing is associated with an existing agent.
* Invalid property data is rejected.
* A nonexistent agent cannot be referenced.
* The created listing is persisted and retrievable.

---

## FR-04 - Retrieve a Listing

The system must allow an individual listing to be retrieved by identifier.

### Acceptance Criteria

* Existing listing returns successfully.
* Nonexistent listing returns a not-found response.
* The response contains the listing and its relevant information.

---

## FR-05 - Update a Listing

The system must allow an existing listing to be updated.

The update operation should support partial updates.

### Acceptance Criteria

* An existing listing can be updated.
* Only supplied fields are changed.
* Invalid values are rejected.
* The listing's update timestamp changes.
* A nonexistent listing returns a not-found response.
* Updating an agent reference must only succeed if the new agent exists.

---

## FR-06 - Delete a Listing

The system must allow a listing to be deleted.

### Acceptance Criteria

* An existing listing can be deleted.
* A deleted listing can no longer be retrieved as an active record.
* Deleting a nonexistent listing returns an appropriate response.

The exact deletion strategy will be determined during the data-model stage.

---

# 4. Listing Search

## FR-07 - List Listings

The system must provide an endpoint for retrieving multiple listings.

The endpoint must support pagination.

The initial search/filter capabilities are:

* property type
* minimum price
* maximum price
* number of bedrooms
* geographic radius

Filters should be combinable.

For example, a consumer should be able to request:

> Sale listings with at least a specified price threshold, two bedrooms, within 10 km of a given location.

---

## FR-08 - Filter by Property Type

The system must allow listings to be filtered by property type.

Supported initial types:

* `rent`
* `sale`
* `shortlet`

The system must reject unsupported property types.

---

## FR-09 - Filter by Price

The system must support:

* minimum price
* maximum price

Both filters may be used independently or together.

### Acceptance Criteria

* Minimum price excludes listings below the requested threshold.
* Maximum price excludes listings above the requested threshold.
* Supplying both returns listings within the requested range.
* Invalid or contradictory price ranges are rejected or handled according to the defined API contract.

---

## FR-10 - Filter by Bedrooms

The system must allow listings to be filtered by bedroom count.

For now we should define whether the supplied bedroom value represents:

* an exact match, or
* a minimum number of bedrooms.

This decision must be made before API implementation because it affects both the API contract and query semantics.

---

## FR-11 - Geographic Radius Search

The system must allow consumers to search for listings within a specified radius of a geographic coordinate.

Inputs:

* latitude
* longitude
* radius

The system must:

* validate that the coordinates are geographically valid
* validate that the radius is positive
* return only listings within the requested distance
* calculate geographic distance using the database's geospatial capabilities

Distance should be expressed in kilometers at the API boundary.

The underlying database representation and geospatial implementation will be determined during the data-model and geospatial-design stages.

---

## FR-12 - Combine Search Filters

Search filters must be composable.

For example:

```text
type = rent
minPrice = 50000
maxPrice = 150000
bedrooms = 2
latitude = ...
longitude = ...
radiusKm = 5
```

The system must apply all supplied filters rather than treating them as mutually exclusive search modes.

---

# 5. Pagination

## FR-13 - Paginate Listing Results

The listing collection endpoint must support pagination.

The API must allow the consumer to specify the requested page and page size, subject to reasonable limits.

The response should provide enough metadata for the consumer to understand the result set.

At minimum, this should include:

* current page
* page size
* total matching records
* total pages

The pagination implementation will be selected during the API and database-design stages.

---

# 6. Validation

## FR-14 - Validate Input

All externally supplied input must be validated before reaching business logic or persistence.

Validation applies to:

* request bodies
* path parameters
* query parameters

Examples of invalid input include:

* missing required fields
* invalid property type
* negative price
* negative bedroom count
* invalid latitude
* invalid longitude
* invalid radius
* invalid pagination parameters
* malformed identifiers

Validation failures must produce a consistent client-error response.

---

# 7. Error Handling

## FR-15 - Consistent Error Responses

The API must return consistent error responses.

Errors should distinguish between at least:

* invalid request
* resource not found
* resource conflict where applicable
* unexpected server failure

Error responses should not expose internal implementation details such as:

* SQL queries
* stack traces
* database credentials
* internal infrastructure information

The final error contract will be defined during API design.

---

# 8. Data Integrity

## FR-16 - Referential Integrity

Every listing must reference an existing agent.

The system must prevent a listing from referencing an agent that does not exist.

The database should enforce this invariant rather than relying exclusively on application-level validation.

---

## FR-17 - Domain Invariants

The system must enforce valid values for core listing attributes.

Examples:

* price must be greater than zero
* bedrooms cannot be negative
* supported listing types are limited to the defined set
* geographic coordinates must be valid
* required fields cannot be null

The final division between application-level validation and database-level constraints will be decided during database design.

---

# 9. Ordering

## FR-18 - Deterministic Listing Ordering

Listing collection responses must have a deterministic ordering.

The default ordering should be defined before implementation.

Possible ordering strategies include:

* newest listings first
* price
* geographic distance when a radius search is supplied

The initial implementation should avoid exposing unnecessary sorting options unless they serve a clear requirement.

---

# 10. API Behavior

## FR-19 - HTTP Semantics

The API should use conventional HTTP semantics.

Examples:

* `POST` for creation
* `GET` for retrieval
* `PATCH` for partial updates
* `DELETE` for deletion

Successful responses should use appropriate HTTP status codes.

Client errors should use appropriate 4xx responses.

Unexpected failures should result in 5xx responses.

The exact API contract will be defined in the API-design stage.

---

# 11. Testing Requirements

## FR-20 - Automated Tests

The system must have automated tests covering important behavior.

At minimum, tests should cover:

### Listing management

* creating a listing
* retrieving a listing
* updating a listing
* deleting a listing

### Validation

* invalid listing data
* invalid search parameters

### Referential integrity

* creating a listing with an invalid agent

### Search

* property type filtering
* price filtering
* bedroom filtering
* combined filters

### Geospatial search

* listings inside the requested radius
* listings outside the requested radius

### Pagination

* page size
* page number
* total results
* total pages

Tests should use a real PostgreSQL/PostGIS database for behavior that depends on database or geospatial functionality.

---

# 12. Operational Requirements

## NFR-01 - Reproducible Local Environment

A developer should be able to start the required local infrastructure with a documented command.

The initial goal is:

```bash
docker compose up
```

The exact container architecture will be determined during the Docker stage.

---

## NFR-02 - Automated Quality Checks

The repository must automatically verify:

* linting
* TypeScript compilation/type checking
* automated tests
* application build

These checks should run through GitHub Actions.

---

## NFR-03 - Configuration Management

Environment-specific configuration must not be hardcoded into source code.

Examples include:

* database connection information
* ports
* environment names
* secrets

The application should validate required configuration at startup.

---

## NFR-04 - Security

The initial API should:

* validate external input
* use parameterized database queries
* avoid exposing sensitive internal errors
* avoid committing secrets
* follow least-privilege principles where applicable

Authentication and authorization are outside the v1 scope.

---

## NFR-05 - Maintainability

The codebase should have clear boundaries betweeb x and n:

* HTTP/APIconcerns
* business logic
* data access
* validation
* infrastructure

The architecture should remain understandable to another engineer joining the project.

---

## NFR-06 - Observability

V1 should provide basic application logging sufficient to diagnose common failures.

Advanced observability such as distributed tracing and centralized log aggregation is outside the v1 scope.

---

## NFR-07 - Performance

The system should use appropriate database indexes for its expected query patterns.

Geospatial queries should use an appropriate spatial index.

The initial implementation does not require benchmarking against production-scale traffic, but query behavior should be examined using realistic test data where appropriate.

---

## C-02 - Technology

The initial implementation will use:

* TypeScript
* Node.js
* Fastify
* PostgreSQL
* PostGIS
* Zod
* Vitest
* Docker
* GitHub Actions

---

## C-03 - Architecture

The initial system will be a modular monolith.

Microservices are not required.

---

## C-04 - Database Access

The project will use SQL directly through a thin database-access layer.

A heavy ORM will be introduced for the v2.

---

# 14. Out of Scope

The following are explicitly excluded from the current requirements:

* authentication
* authorization
* user registration
* bookings
* reservations
* availability calendars
* payments
* subscriptions
* messaging
* notifications
* media storage
* image processing
* utilities
* maintenance
* multi-tenancy
* analytics
* administrative dashboard
* Elasticsearch
* Meilisearch
* Redis
* Kafka
* Kubernetes
* microservices

These may become requirements in future versions but are not requirements of the current system.

---

# 15. Future Requirements Context

The system may eventually evolve into a property-management platform.

Future requirements may include:

### Property Management

* properties
* units
* amenities
* ownership
* property status

### Short-Term Rentals

* bookings
* availability
* guests
* check-in/check-out
* pricing

### Operations

* utilities
* meter readings
* maintenance
* expenses

### Financial

* payments
* invoices
* subscriptions
* owner payouts

### Communication

* messaging
* email
* SMS
* push notifications

### Discovery

* full-text search
* fuzzy search
* advanced geographic search
* recommendations

These future requirements are recorded only as architectural context. They must not become implicit requirements of v1.

---

# 16. Requirements That Need Decisions

The following questions remain intentionally unresolved and must be decided during subsequent design stages:

1. Should `bedrooms` search mean exact match or minimum bedrooms?
2. What should the default listing ordering be?
3. Should deleted listings be permanently deleted or soft-deleted?
4. What pagination strategy should be used?
5. What identifier strategy should be used?
6. Should location be represented as PostGIS `geography` or `geometry`?
7. Which fields require database indexes?
8. What exact API error contract should be exposed?
9. What constitutes a valid listing lifecycle/status?
10. Which invariants belong in the database versus the application?

These are design decisions rather than requirements and should not be prematurely encoded into the implementation.

---

# 17. Requirements Traceability

Each significant requirement should eventually map to implementation and tests.

For example:

| Requirement              | Implementation            | Test                            |
| ------------------------ | ------------------------- | ------------------------------- |
| FR-03 Create Listing     | Listings API + repository | Create listing integration test |
| FR-09 Price Filter       | Listing search query      | Price filter test               |
| FR-11 Radius Search      | PostGIS query             | Geospatial integration test     |
| FR-16 Agent Integrity    | Foreign key               | Invalid agent test              |
| FR-13 Pagination         | Listing query/API         | Pagination test                 |
| NFR-02 CI                | GitHub Actions            | CI workflow                     |
| NFR-01 Local Environment | Docker Compose            | Local setup verification        |

This traceability will help ensure that implementation does not drift away from the actual requirements.
m