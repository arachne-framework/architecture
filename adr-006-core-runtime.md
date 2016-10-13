# Architecture Decision Record: Core Runtime

## Context

At some point, every Arachne application needs to start; to bootstrap
itself from a static project or deployment artifact, initialize what
needs initializing, and begin servicing requests, connecting to
databases, processing data, etc.

There are several logically inherent subtasks to this bootstrapping process, which can be broken down as follows.

- Starting the JVM
    - Assembling the project's dependencies
    - Building a JVM classpath
    - Starting a JVM
- Arachne Specific
    - Reading the initial user-supplied configuration (i.e, the configuration scripts from [ADR-005](adr-005-user-facing-config.md))
    - Initializing the Arachne configuration given a project's set of modules (described in [ADR-002](adr-002-configuration.md) and [ADR-004](adr-004-module-loading.md))
- Application Specific
    - Instantiate user and module-defined objects that needs to exist at runtime.
    - Start and stop user and module-defined services

As discussed in [ADR-004](adr-004-module-loading.md), tasks in the "starting the JVM" category are not in-scope for Arachne; rather, they are offloaded to whatever build/dependency tool the project is using (usually either [boot](http://boot-clj.com) or [leiningen](http://leiningen.org).)

This leaves the Arachne and application-specific startup tasks. Arachne should provide an orderly, structured startup (and shutdown) procedure, and make it possible for modules and application authors to hook into it to ensure that their own code initializes, starts and stops as desired.

Additionally, it must be possible for different system components to have dependencies on eachother, such that when starting, services start *after* the services upon which they depend. Stopping should occur in reverse-dependency order, such that a service is never in a state where it is running but one of its dependencies is stopped.

## Decision

#### Components

Arachne uses the [Component](https://github.com/stuartsierra/component) library to manage system components. Instead of requiring users to define a component system map manually, however, Arachne itself builds one based upon the Arachne config via *Configuration Entities* that appear in the configuration.

Component entities may be added to the config directly by end users (via a initialization script as per [ADR-005](adr-005-user-facing-config.md)), or by modules in their `configure` function ([ADR-004](adr-004-module-loading.md).)

Component entities have attributes which indicates which other components they depend upon. Circular dependencies are not allowed; the component dependency structure must form a Directed Acyclic Graph (DAG.) The dependency attributes also specify the key that Component will use to `assoc` dependencies.

Component entities also have an attribute that specifies a *component constructor function* (via a fully qualified name.) Component constructor functions must take two arguments: the configuration, and the entity ID of the component that is to be constructed. When invoked, a component constructor must return a runtime component object, to be used by the Component library. This may be any object that implements `clojure.lang.Associative`, and may also optionally satisfy Component's `Lifecycle` protocol.

#### Arachne Runtime

The top-level entity in an Arachne system is a reified *Arachne Runtime* object. This object contains both the Component system object, and the configuration value upon which the runtime is based. It satisfies the `Lifecycle` protocol itself; when it is started or stopped, all of the component objects it contains are started or stopped in the appropriate order.

The constructor function for a Runtime takes a configuration value and some number of "roots"; entity IDs or lookup refs of Component entities in the config. Only these root components and their transitive dependencies will be instantiated or added to the Component system. In other words, only component entities that are actually used will be instantiated; unused component entities defined in the config will be ignored.

A `lookup` function will be provided to find the runtime object instance of a component, given its entity ID or lookup ref in the configuraiton.

#### Startup Procedure

Arachne will rely upon an external build tool (such as boot or leiningen.) to handle downloading dependencies, assembling a classpath, and starting a JVM.

Once JVM with the correct classpath is running, the following steps are required to yield a running Arachne runtime:

1. Determine a set of modules to use (the "active modules")
2. Build a configuration schema by querying each active module using its `schema` function ([ADR-004](module-loading.md))
3. Update the config with initial configuration data from user init scripts ([ADR-005](adr-005-user-facing-config.md))
4. In module dependency order, give each module a chance to query and update the configuration using its `configure` function ([ADR-004](module-loading.md))
5. Create a new Arachne runtime, given the configuration and a set of root components.
6. Call the runtime's `start` method. 

The Arachne codebase will provide entry points to automatically perform these steps for common development and production scenarios. Alternatively, they can always be be executed individually in a REPL, or composed in custom startup functions.
 
## Status

PROPOSED

## Consequences


- It is possible to fully define the system components and their dependencies in an application's configuration. This is how Arachne achieves dependency injection and inversion of control.
- It is possible to explicitly create, start and stop Arachne runtimes.
- Multiple Arachne runtimes may co-exist in the same JVM (although they may conflict and fail to start if they both attempt to use a global resource such as a HTTP port)
- By specifying different root components when constructing a runtime, it is possible to run different types of Arachne applications based on the same Arachne configuration value.
