# Property Listings Platform

## 1. Problem Statement

Property seekers often have to search across fragmented property listings, communicate with multiple agents, and manually determine whether properties meet their requirements.

Property owners and agents similarly need a reliable way to publish and manage property listings and make those listings discoverable based on relevant criteria such as property type, price, number of bedrooms, and geographic location.

The initial system will provide a backend API for creating, managing, and discovering property listings.

For the assessment, the system will focus narrowly on the listing and discovery problem: storing property listings and allowing consumers to search for listings using structured filters and geographic proximity.

The system should provide a clean foundation that can later evolve into a broader property-management platform without prematurely implementing functionality that is outside the current problem.

---

## 2. Initial User Problem

A **property seeker** should be able to answer questions such as:

- What properties are available for rent?
- Which properties are within a particular price range?
- Which properties have a particular number of bedrooms?
- Which properties are within X kilometers of a specific location?
- What listings satisfy several of these conditions simultaneously?

An **agent** should be able to:

- Create a property listing.
- View a listing.
- Update a listing.
- Remove a listing.
- Have listings associated with an identifiable agent.

The API is the initial product surface through which these capabilities are provided.

---

## 3. Initial System Goal

Build a backend service that provides:

1. Reliable persistence for property listings.
2. CRUD operations for listings.
3. Structured listing search.
4. Geographic proximity search.
5. Pagination.
6. Input validation.
7. Consistent error handling.
8. Automated tests.
9. A reproducible local development environment.
10. A foundation suitable for future evolution.

---

## 4. Users and Actors

### Property Seeker

Consumes listing data and searches for properties matching desired criteria.

For the assessment, the property seeker is represented only through API requests. Authentication and user accounts are outside the initial scope.

### Agent

Owns or manages property listings.

An agent must exist as a real entity in the database so that listings can maintain referential integrity.

### API Consumer

A web or mobile application will eventually consume the API.

The assessment does not require implementing a frontend or mobile client.

---

## 5. Core Domain

The initial domain contains two primary entities.

### Agent

Represents a person or organization responsible for one or more property listings.

### Listing

Represents a property advertisement made available through the platform.

### Relationship

```
Agent ──owns/manages──▶ many Listings
```

A Listing belongs to exactly one Agent in the initial model.

---

## 6. Core Listing Information

A listing must contain enough information to support the initial discovery experience.

Required concepts include:

- title
- price
- property type
- number of bedrooms
- geographic location
- associated agent

Additional descriptive information may be included where useful, but the assessment should not expand into a complete property-management domain.

---

## 7. Search Problem

The system must support combining structured filters with geographic proximity.

A consumer should be able to ask for listings such as:

> Find rental properties with at least/in a specified number of bedrooms, within a specified price range, and within 5 km of a given coordinate.

The search system must therefore support:

- property type filtering
- minimum price
- maximum price
- bedroom filtering
- geographic radius filtering
- pagination

The geographic search must use the database's geospatial capabilities rather than calculating distances in application code.

---

## 8. Success Criteria

The initial system is successful if it meets the following criteria.

### Correctness

A listing can be created, retrieved, updated and deleted correctly.

### Search

The API returns only listings satisfying the requested filters.

### Geospatial Correctness

A radius search correctly determines whether a listing falls within the requested geographic distance.

### Data Integrity

Invalid data cannot be persisted.

A listing cannot reference a nonexistent agent.

### Reliability

Invalid requests produce useful errors rather than application crashes or ambiguous responses.

### Testability

Important business and database behavior can be verified automatically.

### Reproducibility

Another developer should be able to clone the repository and run the application locally using the documented setup.

### Extensibility

The design should allow the system to evolve without requiring the initial listing functionality to be rewritten.

---

## 9. Constraints

The assessment has a deliberately small scope and a limited implementation time.

The initial implementation should therefore favor:

- simplicity
- correctness
- maintainability
- clear boundaries
- production-minded practices

over premature scalability or infrastructure complexity.

The initial system will use:

| Concern          | Technology      |
| ---------------- | --------------- |
| Language         | TypeScript      |
| Runtime          | Node.js         |
| Web framework    | Fastify         |
| Database         | PostgreSQL      |
| Geospatial       | PostGIS         |
| Validation       | Zod             |
| Testing          | Vitest          |
| Containerization | Docker          |
| CI               | GitHub Actions  |

The system will initially be implemented as a **modular monolith**.

---

## 10. Explicitly Out of Scope

The following are not part of the assessment:

- authentication
- authorization
- user accounts
- property reservations
- booking management
- payments
- subscriptions
- messaging
- notifications
- media/image management
- utilities management
- maintenance management
- multi-tenancy
- administrative UI
- full-text search engines
- Elasticsearch
- Meilisearch
- Redis
- Kafka
- Kubernetes
- microservices

These may become relevant in future iterations but should not influence the initial implementation beyond reasonable architectural considerations.

---

## 11. Future Product Direction

The listing API is intended to be the first stage of a broader property platform.

A future version may support property owners and short-term-rental operators managing their properties.

Potential future capabilities include:

- property management
- multiple units per property
- reservations
- availability calendars
- guest management
- utility management
- meter readings
- maintenance requests
- payments
- subscriptions
- notifications
- messaging
- search and discovery
- reporting and analytics

These future capabilities are not requirements for the current system. They provide architectural context rather than implementation requirements.

---

## 12. Architectural Principle

The initial system should be designed for **evolution, not prediction**.

We should establish clear domain boundaries and preserve data integrity without implementing infrastructure or abstractions for hypothetical future requirements.

Future complexity should be introduced when an actual requirement justifies it.

---

## 13. Key Questions for the Next Design Stage

Before implementation, the following questions need to be answered:

1. What exactly constitutes an Agent?
2. What exactly constitutes a Listing?
3. What invariants must the database enforce?
4. What API operations are required?
5. How should geographic coordinates be represented?
6. How should search queries be modeled?
7. How should pagination work?
8. Which database indexes are required?
9. What failure cases must the API handle?
10. What parts of the system require integration testing?

These questions will be addressed in subsequent design stages rather than being prematurely decided during implementation.