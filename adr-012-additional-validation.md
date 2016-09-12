# Architecture Decision Record: Enhanced Validation

## Context

As much as possible, an Arachne application should be defined by its configuration. If something is wrong with the configuration, there is no way that an application can be expected to work correctly.

Therefore, it is desirable to validate that a configuration is correct to the greatest extent possible, at the earliest possible moment. This is important for two distinct reasons:

- Ease of use and developer friendliness. Config validation can return helpful errors that point out exactly what's wrong instead of deep failures with lengthy debug sessions.
- Program correctness. Some types of errors in configs might not be discovered at all during testing or development, and aggressively failing on invalid configs will prevent those issues from affecting end users in production.

There are two "kinds" of config validation.

The first is ensuring that a configuration as data is structurally correct; that it adheres to its own schema. This includes validating types and cardinalities as expressed by the Arachne's core ontology system.

The second is ensuring that the Arachne Runtime constructed from a given configuration is correct; that the runtime component instances returned by component constructors are of the correct type and likely to work.

## Decision

Arachne will perform both kinds of validation. To disambiguate them (since they are logically distinct), we will term the structural/schema validation "configuration validation", while the validation of the runtime objects will be "runtime validation."

Both styles of validation should be extensible by modules, so modules can specify additional validations, where necessary.

#### Configuration Validation

Configuration validation is ensuring that an Arachne configuration object is consistent with itself and with its schema.

Because this is ultimately validating a set of Datomic style `eavt` tuples, the natural form for checking tuple data is Datalog queries and query rules, to search for and locate data that is "incorrect." 

Each logical validation will have its own "validator", a function which takes a config, queries it, and either returns or throws an exception. To validate a config, it is passed through every validator as the final step of building a module.

The set of validators is open, and defined in the configuration itself. To add new validators, a module can transact entities for them during its configuration building phase.

#### Runtime Validation

Runtime validation occurs after a runtime is instantiated, but before it is started. Validation happens on the component level; each component may be subject to validation.

Unlike Configuration validation, Runtime validation uses Spec. What specs should be applied to each component are defined in the configuration using a keyword-valued attribute. Specs may be defined on individual component entities, or to the *type* of a component entity. When a component is validated, it is validated using all the specs defined for it or any of its supertypes.

## Status

PROPOSED

## Consequences

- Validations have the opportunity to find errors and return clean error messages
- Both the structure of the config and the runtime instances can be validated
- The configuration itself describes how it will be validated
- Modules have complete flexibility to add new validations
- Users can write custom validations


