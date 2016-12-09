# Architecture Decision Record: Simplification of Chimera Model

Note: this ADR supersedes some aspects of [ADR-15](adr-015-data-abstraction-model.md) and [ADR-16](adr-016-db-migrations.md).

## Context

The Chimera data model (as described in ADR-15 and ADR-16) includes the concepts of _entity types_ in the domain data model: a defined entity type may have supertypes, and inherits all the attributes of a given supertype 

This is quite expressive, and is a good fit for certain types of data stores (such as Datomic, graph databases, and some object stores.) It makes it possible to compose types, and re-use attributes effectively.

However, it leads to a number of conceptual problems, as well as implementation complexities. These issues include but are not limited to:

- There is a desire for some types to be "abstract", in that they exist purely to be extended and are not intented to be reified in the target database (e.g, as a table.) In the current model it is ambiguous whether this is the case or not.
- A singe `extend-type` migration operation may need to create multiple columns in multiple tables, which some databases do not support transactionally.
- When doing a lookup by attribute that exists in multiple types, it is ambiguous which type is intended.
- In a SQL database, how to best model an extended type becomes ambiguous: copying the column leads to "denormalization", which might not be desired. On the other hand, creating a separate table for the shared columns leads to more complex queries with more joins.

All of these issues can be resolved or worked around. But they add a variable amount of complexity cost to every Chimera adapter, and create a domain with large amounts of ambigous behavior that must be resolved (and which might not be discovered until writing a particular adapter.)

## Decision

The concept of type extension and attribute inheritance does not provide benefits proportional to the cost.

We will remove all concept of supertypes, subtypes and attribute inheritance from Chimera's data model.

Chimera's data model will remain "flat". In order to achieve attribute reuse for data stores for which that is idiomatic (such as Datomic), multiple Chimera attributes can be mapped to a single DB-level attribute in the adapter mapping metadata.
   
## Status

PROPOSED

## Consequences

- Adapters will be significantly easier to implement.
- An attribute will need to be repeated if it is present on different domain entity types, even if it is semantically similar.
- Users may need to explicitly map multiple Chimera attributes back to the same underlying DB attr/column if they want to maintain an idiomatic data model for their database.



