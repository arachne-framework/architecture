# Architecture Decision Record: Configuration Ontology

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

- We will add meta-attributes to the schema of every configuration, expressing basic ontological relationships.
- These attributes will be semantically compatible with OWL (such that we could conceivably in the future generate an OWL ontology from a config schema)
- The initial set of these attributes will be minimal, and targeted towards the information necessary to generate rich schema diagrams
  - classes and superclass
  - attribute domain
  - attribute range (for ref attributes)
  - min and max cardinality
- Arachne core will provide some (optional) utility functions for schema generation, to make writing module schemas less verbose.

## Status

PROPOSED

## Consequences

- Arachne schemas will reify the concept of entity type and the possible relationships between entities of various types.
- We will have an approach for adding additional semantic attributes in the future, as it makes sense to do so.
- We will not be obligated to define an entire ontology up front
- Modules usage of the defined ontology is not technically enforced. Some, (such as entity type relationships) will be the strong convention and possibly required for tool support; others (such as min and max cardinality) will be optional.
- We will preserve the possibility for interop with OWL in the future.
