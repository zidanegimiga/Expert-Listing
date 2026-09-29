# Domain Model

## 1. Purpose

This document defines the core concepts, relationships, boundaries, and business rules of the Property Listings API.

The domain model describes what exists in the system and how those concepts relate to one another. It intentionally avoids implementation-specific decisions such as database tables, indexes, SQL queries, HTTP routes, or framework structure.

The goal is to establish a stable conceptual model that can support the current version while leaving room for the system to evolve into a broader property-management platform.

---

## 2. Domain Vocabulary

The initial system contains two primary domain entities:

* **Agent**
* **Listing**

The system also contains several concepts that belong to a listing but do not currently need to exist as independent entities:

* Property type
* Price
* Location
* Bedroom count

### Agent

An Agent is a person or business representative responsible for publishing and managing property listings through the system.

For the scope of this version, an Agent has a minimal identity and is primarily associated with the listings they manage.

V1 does not require authentication, authorization, agent profiles, or agent-specific business workflows.

### Listing

A Listing is a marketplace representation of a property made available through an agent.

A listing contains the information required for a potential property seeker to discover and understand the offering, including:

* Title
* Description
* Price
* Property type
* Number of bedrooms
* Geographic location
* Address information
* Agent responsible for the listing

A listing is the primary searchable resource in the system.

---

## 3. Property vs Listing

The current system deliberately does not introduce a separate `Property` entity.

Conceptually, a distinction exists:

**Property**

A physical real-world asset such as an apartment, house, office, or land parcel.

**Listing**

A representation of that property published for discovery or transaction purposes.

A single physical property could eventually have multiple listings over its lifetime or across different channels.

For example:

```text
Physical Property
      │
      ├── Sale Listing
      │
      └── Rental Listing
```

However, introducing this distinction into the current version would add complexity that is not required by the specification.

Therefore, the current model treats the Listing as the primary domain entity.

This is an intentional scope decision rather than an assertion that Property and Listing are inherently the same concept.

A future property-management system can introduce `Property` as a separate entity when the requirements justify it.

---

## 4. Relationships

### Agent → Listing

An Agent can manage multiple Listings.

Each Listing belongs to one Agent.

Conceptually:

```text
Agent
  │
  │ manages
  │
  ├──────── Listing
  ├──────── Listing
  └──────── Listing
```

Cardinality:

```text
Agent 1 ─────────── * Listing
```

A listing cannot exist without an associated agent within the current domain model.

The system does not currently support multiple agents jointly owning or managing a single listing.

---

## 5. Listing Concepts

### Property Type

A listing has one supported property transaction type:

* `rent`
* `sale`
* `shortlet`

The type describes how the listing is being offered.

It is a constrained domain value rather than arbitrary free-form text.

### Price

A listing has a price representing the monetary amount associated with the offering.

The current version only requires a numeric price.

Currency handling is intentionally kept simple at this stage. If the product later supports multiple currencies or financial workflows, money can become a richer domain concept containing amount and currency.

### Location

A listing has a geographic location represented conceptually by:

```text
latitude
longitude
```

The location is required because geographic proximity is a core search capability of the system.

Latitude and longitude together represent a single location rather than two unrelated pieces of information.

### Bedrooms

A listing has a number of bedrooms.

The value represents a count and therefore cannot conceptually be negative.

---

## 6. Domain Invariants

An invariant is a rule that should remain true whenever an entity exists in a valid domain state.

The current domain invariants include:

### Listing invariants

1. A listing must have a title.
2. A listing must have a valid property type.
3. A listing must have a valid price.
4. A listing must have a valid bedroom count.
5. A listing must have a valid geographic location.
6. A listing must belong to an agent.
7. A listing cannot reference an agent that does not exist.
8. Latitude must represent a valid geographic latitude.
9. Longitude must represent a valid geographic longitude.
10. A listing's price cannot be negative.
11. A listing's bedroom count cannot be negative.

These rules describe domain correctness.

The implementation mechanism used to enforce them will be decided later.

---

## 7. Search Rules

Search operates primarily over Listings.

The domain supports filtering by:

* Property type
* Minimum price
* Maximum price
* Number of bedrooms
* Geographic proximity

Filters may be combined.

For example:

```text
Find rental listings
under a specified price
with at least or exactly a specified number of bedrooms
within a specified distance of a geographic point.
```

The exact interpretation of ambiguous filter semantics, such as whether a bedroom filter means an exact count or a minimum count, must be finalized before implementation.

---

## 8. Geographic Search Concept

Geographic search is a domain capability rather than a database feature.

The domain concept is:

> Find listings whose location falls within a specified distance of a given geographic point.

For example:

```text
Search point
     ●
     │
     │ radius
     ↓
   (     )
  (  ● ●  )
   ( ● ● )
```

The domain does not care whether this is eventually implemented using PostGIS, another database, or a dedicated search system.

The persistence and query implementation will be defined in later design stages.

---

## 9. Listing Lifecycle

The current version does not require a complex listing lifecycle.

A listing can be:

```text
Created → Available through the API → Updated → Deleted
```

The version does not currently require concepts such as:

* Draft
* Published
* Archived
* Suspended
* Sold
* Rented
* Expired

These may become meaningful in a production marketplace, but introducing them now would add business rules that are not required by the version.

The absence of a lifecycle status is therefore a deliberate scope decision.

---

## 10. Aggregate Boundaries

For the current system, `Listing` is the primary aggregate for listing operations.

An Agent is referenced by a Listing, but the Listing does not own or contain the Agent.

Conceptually:

```text
Agent

Listing
 ├── title
 ├── price
 ├── type
 ├── bedrooms
 ├── location
 └── agent reference
```

The relationship between the two entities will be enforced through the system's persistence model.

This keeps the current domain simple while preserving the possibility of richer agent capabilities later.

---

## 11. Domain Rules vs API Rules

Not every validation rule belongs to the domain.

For example:

### Domain rule

```text
A listing cannot have a negative price.
```

This should remain true regardless of how a listing is created.

### API rule

```text
POST /listings must contain a JSON request body.
```

This is an HTTP/API concern.

### Persistence rule

```text
agent_id must reference an existing agent.
```

The underlying domain relationship requires this, while the database can enforce it using referential integrity.

Keeping these concerns separate prevents implementation details from leaking into the domain model.

---

## 12. Current Domain Model

The resulting conceptual model is:

```text
                    ┌───────────────┐
                    │     Agent     │
                    └───────┬───────┘
                            │
                         manages
                            │
                            │ 1
                            │
                            │
                            │ *
                    ┌───────▼───────┐
                    │    Listing    │
                    ├───────────────┤
                    │ title         │
                    │ description   │
                    │ price         │
                    │ type          │
                    │ bedrooms      │
                    │ location      │
                    │ address       │
                    │ agent         │
                    └───────────────┘
```

---

## 13. Future Evolution

The current model intentionally leaves room for a broader property-management platform.

A future model may introduce concepts such as:

```text
Property
 ├── Unit
 ├── Owner
 ├── Listing
 ├── Reservation
 ├── Guest
 ├── Maintenance Request
 ├── Meter Reading
 └── Payment
```

The important architectural principle is:

> Design the current system so that these concepts can be introduced later without unnecessarily implementing them today.

The current version therefore optimizes for a small, coherent domain rather than attempting to model the entire future platform.

---

## 14. Open Domain Questions

The following questions should be resolved before the corresponding implementation decisions are finalized:

1. Does `bedrooms` represent an exact match or a minimum number?
2. Should price have an explicit currency?
3. Should listings eventually have a lifecycle/status?
4. Should a deleted listing be permanently removed or retained?
5. When should `Property` become a separate entity?
6. Does the system eventually need multiple agents per listing?
7. What additional property information becomes necessary when the platform expands into property management?

These questions are deliberately separated from implementation decisions. They should only be resolved when the relevant product requirement requires them.

---

## 15. Design Principle

The domain model follows a simple principle:

> **Model what the current requirements require, while avoiding decisions that make future evolution unnecessarily difficult.**

The system should be easy to extend, but the version should remain small enough to understand, test, and deliver within its time constraints.
