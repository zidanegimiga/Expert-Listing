# Domain Model

## 1. Purpose

This document defines the core concepts, relationships, boundaries, and business rules of the Property Platform.

The domain model describes what exists in the system and how those concepts relate to one another. It intentionally avoids implementation-specific decisions such as database tables, SQL queries, HTTP routes, framework structure, or infrastructure.

The first version focuses on users, roles, agents, listings, and listing discovery while leaving room for the platform to evolve into a broader property-management system.

---

# 2. Core Domain Concepts

The initial domain contains four important concepts:

* **User**
* **Role**
* **Agent Profile**
* **Listing**

These concepts have different responsibilities and should not be treated as interchangeable.

### User

A User represents an account or identity within the platform.

A user answers:

> **Who is this person or account?**

A user may interact with the platform in different capacities.

For example:

```text
User
 ├── Customer
 └── Agent
```

A user is therefore not inherently an agent or customer.

Those are roles or capabilities associated with the user.

---

### Role

A Role represents a set of capabilities or permissions associated with a user.

Initial roles may include:

* `customer`
* `agent`
* `admin`

Additional roles can be introduced as the product evolves.

A user may have more than one role.

For example:

```text
Jane
 ├── customer
 └── agent
```

This allows someone to search for properties personally while also managing listings professionally.

Roles should not be used to store domain-specific information.

A role answers:

> **What can this user do?**

It does not answer:

> **What information does this user have?**

---

### Agent Profile

An Agent Profile represents information specific to a user acting as an agent.

The distinction is:

```text
User
 └── identity

Agent Profile
 └── agent-specific business information
```

A user can therefore have an Agent Profile when they operate as an agent.

The first version may keep the Agent Profile minimal because the current product does not require extensive agent-management functionality.

Future agent-specific information could include:

* Agency
* License information
* Professional contact information
* Verification status
* Biography
* Areas of operation

This information should not be placed directly into the general User identity when it is only meaningful to agents.

---

### Listing

A Listing is a marketplace representation of a property made available for discovery through an agent.

A listing contains information such as:

* Title
* Description
* Price
* Property type
* Number of bedrooms
* Geographic location
* Address information
* The agent responsible for the listing

A Listing is the primary searchable resource in the first version.

---

# 3. Identity vs Role vs Domain Profile

These concepts must remain distinct.

Consider a person named Jane.

```text
User
├── name: Jane
├── email: jane@example.com
└── phone: ...
```

Jane may have:

```text
Roles
├── customer
└── agent
```

And because she is an agent:

```text
Agent Profile
├── agency
├── license information
└── verification status
```

The concepts answer different questions:

| Concept       | Question answered                                    |
| ------------- | ---------------------------------------------------- |
| User          | Who is this?                                         |
| Role          | What can this user do?                               |
| Agent Profile | What agent-specific information does this user have? |
| Listing       | What property offering is being published?           |

This separation prevents the system from assuming that being an agent makes someone a fundamentally different type of user.

---

# 4. User and Listing Relationship

A Listing is managed by an agent.

Because an agent is a user with the agent capability, the conceptual relationship is:

```text
User
 │
 │ has agent capability
 ▼
Agent Profile
 │
 │ manages
 ▼
Listing
```

Cardinality:

```text
User 1 ─── 0..1 Agent Profile
                    │
                    │ 1
                    │
                    │
                    │ *
                    ▼
                  Listing
```

A user may have no Agent Profile or one Agent Profile.

An Agent Profile may manage multiple Listings.

A Listing has one responsible agent in the initial product.

---

# 5. Why Not Create a Customer Entity?

Customer is initially modeled as a role rather than a separate domain entity.

A person does not need a `Customer` record simply because they are allowed to search for or interact with listings.

For example:

```text
User
 └── role: customer
```

is sufficient when the platform only needs to know that the user has customer capabilities.

A separate Customer domain entity becomes appropriate when customers acquire meaningful domain-specific state.

For example, a future platform might need:

```text
Customer
├── saved properties
├── viewing preferences
├── booking history
├── verification state
├── payment profile
└── communication preferences
```

At that point, a Customer profile or entity can be introduced without changing the fundamental User identity model.

---

# 6. Property vs Listing

The system deliberately distinguishes the conceptual idea of a physical Property from a Listing.

### Property

A Property is a physical real-world asset.

Examples:

* Apartment
* House
* Office
* Land parcel
* Vacation home

### Listing

A Listing is a representation of a property published for discovery or transaction.

A property could eventually have multiple listings.

For example:

```text
Property
   │
   ├── Sale Listing
   │
   └── Rental Listing
```

A property could also eventually appear through different channels.

However, the first version does not require a separate Property entity.

The distinction is documented now because it is important to the future domain model.

We should not prematurely introduce the entity until the product needs to manage physical properties independently from their listings.

---

# 7. Current Conceptual Model

The current model is therefore:

```text
                         ┌──────────────┐
                         │     User     │
                         └──────┬───────┘
                                │
                     ┌──────────┴──────────┐
                     │                     │
                has roles             may have
                     │                     │
          ┌──────────┼──────────┐          ▼
          ▼          ▼          ▼    ┌──────────────┐
      Customer      Agent      Admin  │ Agent Profile│
                                      └──────┬───────┘
                                             │
                                          manages
                                             │
                                             ▼
                                      ┌──────────────┐
                                      │   Listing    │
                                      └──────────────┘
```

This model keeps identity separate from business capabilities.

---

# 8. Listing Concepts

## Property Type

A Listing has one supported offering type:

* `rent`
* `sale`
* `shortlet`

This is a constrained domain value.

It should not be treated as arbitrary free-form text.

---

## Price

A Listing has a monetary price associated with its offering.

The initial domain only requires an amount.

Currency is conceptually part of money, even if the first version operates within a single currency.

If the platform later supports multiple currencies, the domain can evolve toward:

```text
Money
├── amount
└── currency
```

The persistence representation will be determined separately.

---

## Location

A Listing has a geographic location.

Conceptually:

```text
Location
├── latitude
└── longitude
```

The location represents a single geographic point.

The first version requires geographic location because proximity search is a core product capability.

---

## Bedrooms

A Listing contains a number of bedrooms.

The value represents a count and therefore cannot conceptually be negative.

---

# 9. Domain Invariants

An invariant is a rule that must remain true whenever the domain is in a valid state.

### User

1. A User represents one platform identity.
2. A User may have multiple roles.
3. Roles are constrained to supported capabilities.

### Agent

1. An Agent Profile belongs to one User.
2. A User can have at most one Agent Profile.
3. A user managing Listings must possess agent capability.
4. An Agent Profile may manage multiple Listings.

### Listing

1. A Listing must have a title.
2. A Listing must have a valid property type.
3. A Listing must have a valid price.
4. A Listing must have a valid bedroom count.
5. A Listing must have a valid geographic location.
6. A Listing must have a responsible agent.
7. The responsible agent must correspond to an existing platform identity.
8. Latitude must represent a valid geographic latitude.
9. Longitude must represent a valid geographic longitude.
10. Price cannot be negative.
11. Bedroom count cannot be negative.

---

# 10. Role Semantics

Roles represent capabilities rather than mutually exclusive identities.

The initial conceptual role set is:

```text
customer
agent
admin
```

A user may possess multiple roles.

For example:

```text
User A
├── customer
└── agent
```

This avoids a model where a person must be classified permanently as either a customer or an agent.

Future roles could include:

```text
property_owner
property_manager
staff
support
finance
```

The actual role set should evolve according to product requirements.

---

# 11. Authorization vs Domain Ownership

Having a role does not automatically mean that a user owns every resource associated with that role.

For example:

```text
User A
└── role: agent
```

does not mean User A can modify every Listing.

The platform may later distinguish:

```text
Role:
Can an agent perform listing operations?

Ownership:
Which specific listings can this agent modify?
```

This distinction becomes important when multiple agents, agencies, property owners, or property managers interact with the same resources.

The first version can keep authorization simple while preserving this conceptual distinction.

---

# 12. Search

Listings are the primary searchable resource.

Search supports:

* Property type
* Minimum price
* Maximum price
* Bedrooms
* Geographic proximity

Filters may be combined.

For example:

```text
Find rental listings
within a specified radius
under a specified price
with a specified bedroom requirement.
```

The exact semantics of ambiguous filters, such as whether bedroom count represents an exact match or minimum requirement, must be finalized before implementation.

---

# 13. Geographic Search

Geographic search is a domain capability.

The domain operation is:

> Find listings whose location falls within a specified distance of a geographic point.

Conceptually:

```text
                 Search point
                      ●
                   ╱     ╲
                 ╱         ╲
               ╱   radius   ╲
             ●      ●        ●
               ╲           ╱
                 ╲       ╱
                   ╲___╱
```

The domain does not dictate how the operation is implemented.

PostGIS is an infrastructure and persistence decision that will be documented later.

---

# 14. Listing Lifecycle

The first version does not require a complex listing lifecycle.

The basic lifecycle is:

```text
Created
   ↓
Available through the platform
   ↓
Updated
   ↓
Deleted
```

The domain does not currently require:

* Draft
* Published
* Archived
* Suspended
* Sold
* Rented
* Expired

These may become valuable once marketplace and property-management workflows become more sophisticated.

They should be introduced when they represent real product states rather than anticipated future complexity.

---

# 15. Aggregate Boundaries

The initial product has several conceptual boundaries.

### User boundary

The User represents platform identity and roles.

### Agent Profile boundary

The Agent Profile contains agent-specific information.

### Listing boundary

The Listing represents a marketplace offering.

The Listing does not contain or own the User or Agent Profile.

Conceptually:

```text
User
  │
  └── Agent Profile
          │
          └── Listing
```

The exact persistence relationships and transaction boundaries will be defined during the data-model and architecture stages.

---

# 16. Domain Rules vs Application Rules

Not every rule belongs to the domain itself.

### Domain rule

```text
A listing cannot have a negative price.
```

This should remain true regardless of how the listing is created.

### Authorization rule

```text
Only a user with agent capability can create an agent listing.
```

This concerns access to an operation.

### HTTP rule

```text
POST /listings requires a JSON request body.
```

This concerns the API transport mechanism.

### Persistence rule

```text
A listing must reference a valid agent profile.
```

The relationship is a domain requirement, while the database may enforce the corresponding referential integrity.

Keeping these concerns separate prevents infrastructure and transport concepts from leaking into the domain.

---

# 17. Future Property-Management Domain

The identity model provides a foundation for a broader platform.

A future domain could look like:

```text
                              User
                                │
             ┌──────────────────┼───────────────────┐
             │                  │                   │
          Customer            Agent          Property Owner
             │                  │                   │
             │                  │                   │
             │                  └── Listings         │
             │                                      │
             └───────────────┐                      │
                             ▼                      ▼
                          Booking              Property
                             │                      │
                           Guest                    │
                                                    ▼
                                                   Unit
```

Additional domains could later include:

```text
Booking
Availability
Payments
Payouts
Maintenance
Utilities
Messaging
Notifications
Search
Analytics
```

The first version does not implement these domains.

The purpose of documenting them is to prevent today's identity and domain model from unnecessarily blocking tomorrow's product.

---

# 18. Important Design Principle

The platform distinguishes three fundamentally different questions:

```text
WHO?
  ↓
User

WHAT CAN THEY DO?
  ↓
Role

WHAT BUSINESS INFORMATION DO THEY HAVE?
  ↓
Domain Profile / Entity
```

For example:

```text
User
  │
  ├── roles: customer, agent
  │
  └── Agent Profile
          │
          └── manages Listings
```

This is more flexible than representing every user category as a separate identity table.

---

# 19. Open Domain Questions

The following questions remain intentionally open until the relevant design stage:

1. Which roles are required in the first version?
2. Does the first version require authentication, or is User currently a domain foundation only?
3. What information belongs in the Agent Profile?
4. Should an Agent Profile be required before a user can create Listings?
5. What exactly does the bedroom filter mean?
6. Should price have an explicit currency?
7. What constitutes ownership of a Listing?
8. Should multiple agents eventually manage one Listing?
9. When should Property become a separate persisted entity?
10. When do customers require a dedicated domain profile?
11. How should role-based authorization interact with resource ownership?

These questions should be resolved according to actual product behavior rather than prematurely encoded into infrastructure.

---

# 20. Final V1 Domain Model

The V1 domain can therefore be summarized as:

```text
                         ┌─────────────┐
                         │    User     │
                         └──────┬──────┘
                                │
                         has one or more
                                │
                                ▼
                         ┌─────────────┐
                         │    Roles    │
                         └──────┬──────┘
                                │
                         agent capability
                                │
                                ▼
                      ┌──────────────────┐
                      │  Agent Profile   │
                      └────────┬─────────┘
                               │
                            manages
                               │
                               ▼
                      ┌──────────────────┐
                      │     Listing      │
                      ├──────────────────┤
                      │ title            │
                      │ description      │
                      │ price            │
                      │ type             │
                      │ bedrooms         │
                      │ location         │
                      │ agent            │
                      └──────────────────┘
```

The model intentionally keeps **identity, capability, and business entities separate**.

That gives the platform a stable foundation for future property-management capabilities without requiring those capabilities to exist in the first version.
