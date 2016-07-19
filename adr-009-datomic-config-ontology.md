# Architecture Decision Record: Abstract Modules

## Context

In [ADR-003](adr-003-config-implementation.md) it was decided to use a Datomic-based configuration, the alternative being something more semantically or ontologically descriptive such as RDF+OWL. 

Although we elected to use Datomic, Datomic does not itself offer much ontological modeling capacity. It has no built-in notion of types/classes, and its attribute specifications are limited to what is necessary for efficient storage and indexing, rather than expressive or validative power.

Ideally, we want modules to be able to communicate additional information about the structure and intent of their domain model, including:

- Types of entities which can exist
- Relationships between those types
- Logical constraints on the values of attributes:
    - more fine grained cardinality; optional/required attributes
    - valid value ranges
    - target entity type (for ref attributes)

This additional data could serve three purposes:

- Documentation about the intended purpose and structure of the configuration defined by a module.
- Deeper, more specific validation of user-supplied configuration values
- Machine-readable integration point for tools which consume and produce Arachne configurations.

## Decision

TBD

## Status

DRAFT

## Consequences

TBD