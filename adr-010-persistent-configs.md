# Architecture Decision Record: Persistent Configuration

## Context

While many Arachne applications will use a transient config which is rebuilt from its initialization scripts every time an instance is started, some users might wish instead to store their config persistently in a full Datomic instance.

There are a number of possible benefits to this approach:

- Deployments from the same configuration are highly reproducible
- Organizations can maintain an immutable persistent log of configuration changes over time.
- External tooling can be used to persistently build and define configurations, up to and including full "drag and drop" architecture or application design.

Doing this introduces a number of additional challenges:

- **Initialization Scripts**: Having a persistent configuration introduces the question of what role initialization scripts play in the setup. Merely having a persistent config does not make it easier to modify by hand - quite the opposite. While an init script could be used to create the configuration, it's not clear how they would be updated from that point (absent a full config editor UI.)
  
  Re-running a modified configuration script on an existing configuration poses challenges as well; it would require that all scripts be idempotent, so as not to create spurious objects on subsequent runs. Also, scripts would then need to support some concept of retraction.
- **Scope & Naming**: It is extremely convenient to use `:db.unique/identity` attributes to identify particular entities in a configuration and configuration init scripts. This is not only convenient, but *required* if init scripts are to be idempotent, since this is the only mechanism by which Datomic can determine that a new entity is "the same" as an older entity in the system.

  However, if there are multiple different configurations in the same database, there is the risk that some of these unique values might be unintentionally the same and "collide", causing inadvertent linkages between what ought to be logically distinct configurations.
 
  While this can be mitigated in the simple case by ensuring that every config uses its own unique namespace, it is still something to keep in mind.

- **Configuration Copying & Versioning** Although Datomic supports a full history, that history is linear. Datomic does not currently support "forking" or maintaining multiple concurrent versions of the same logical data set.

  This does introduce complexities when thinking about "modifying" a configuration, while still keeping the old one. This kind of "fork" would require a deep clone of all the entities in the config, *as well as* renaming all of the `:db.unique/identity` attrs.
  
  Renaming identity attributes compounds the complexity, since it implies that either idents cannot be hardcoded in initialization scripts, or the same init script cannot be used to generate or update two different configurations.

- **Environment-specific Configuration**: Some applications need slightly different configurations for different instances of the "same" application. For instance, some software needs to be told what its own IP address is. While it makes sense to put this data in the configuration, this means that there would no longer be a single configuration, but N distinct (yet 99% identical) configurations.

  One solution would be to not store this data in the configuration (instead picking it up at runtime from an environment variable or secondary config file), but multiplying the sources of configuration runs counter to Arachne's overriding philosophy of putting everything in the configuration to start with.
  
- **Relationship with module load process**: Would the stored configuration represent only the "initial" configuration, before being updated by the active modules? Or would it represent the complete configuration, after all the modules have completed their updates?

  Both alternatives present issues.
  
  If only the user-supplied, initial config is stored, then the usefulness of the stored config is diminished, since it does not provide a comprehensive, complete view of the configuration.
  
  On the other hand, if the complete, post-module config is persisted, it raises more questions. What happens if the user edits the configuration in ways that would cause modules to do something different with the config? Is it possible to run the module update process multiple times on the same config? If so, how would "old" or stale module-generated values be removed?

#### Goals

We need a technical approach with good answers to the challenges described above, that enables a clean user workflow. As such, it is useful to enumerate the specific activities that it would be useful for a persistent config implementation to support:

- Define a new configuration from an init script.
- Run an init script on an existing configuration, updating it.
- Edit an existing configuration using the REPL.
- Edit an existing configuration using a UI.
- Clone a configuration
- Deploy based on a specific configuration

At the same time, we need to be careful not to overly complicate things for the common case; most applications will still use the pattern of generating a configuration from an init script immediately before running an application using it.

## Decision

We will not attempt to implement a concrete strategy for config persistence at this time; it runs the risk of becoming a quagmire that will halt forward momentum.

Instead, we will make a minimal set of choices and observations that will enable forward progress while preserving the ability to revisit the issue of persistent configuration at some point in the future.

1. The configuration schema itself should be compatible with having several configurations present in the same persistent database. Specifically:
  - Each logical configuration should have its own namespace, which will be used as the namespace of all `:db.unique/identity` values, ensuring their global uniqueness.
  - There is a 'configuration' entity that reifies a config, its possible root components, how it was constructed, etc.
  - The entities in a configuration must form a connected graph. That is, every entity in a configuration must be reachable from the base 'config' entity. This is required to have any ability to identify the config as a whole within for any purpose.

2. The current initial _tooling_ for building configurations (including the init scripts) will focus on building configurations from scratch. Tooling capable of "editing" an existing configuration is sufficiently different, with a different set of requirements and constraints, that it needs its own design process.

3. Any future tooling for storing, viewing and editing configurations will need to explicitly determine whether it wants to work with the configuration before or after processing by the modules, since there is a distinct set of tradeoffs.

## Status

PROPOSED

## Consequences

1. We can continue making forward progress on the "local" configuration case.
2. Storing persistent configurations remains possible.
3. It is immediately possible to save configurations for repeatability and debugging purposes.
	- The editing of persistent configs is what will be more difficult.
4. When we want to edit persistent configurations, we will need to analyze the specific use cases to determine the best way to do so, and develop tools specific to those tasks.
