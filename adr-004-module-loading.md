# Architecture Decision Record: Module Structure & Loading

## Context

Arachne needs to be as modular as possible. Not only do we want the
community to be able to contribute new abilities and features that
integrate well with the core and with eachother, we want some of the
basic functionality of Arachne to be swappable for alternatives as
well.

[ADR-002](adr-002-configuration.md) specifies that one role of modules
is to contribute schema to the application config. Other roles of
modules would include providing code (as any library does), and
querying and updating the config during the startup
process. Additionally, since modules can depend upon eachother, they
must specify which modules they depend upon.

Ideally there will be as little overhead as possible for creating and
consuming modules.

Some of the general problems associated with plugin/module systems include:

- Finding and downloading the implementation of the module.
- Discovering and activating the correct set of installed modules.
- Managing module versions and dependencies.

## Decision

Modules will be packaged, distributed and downloaded in standard Maven artifacts.

Modules that have a dependency on another module must specify a Maven
dependency on the artifact containing the module.

Modules will provide no versioning system beyond what Maven provides.

Each module JAR will contain a special `arachne-modules.edn` file at
the root of its classpath. This data file (when read) contains a
sequence of *module definition maps*.

Each module definition map contain the following information:

- The formal name of the module (as a namespaced symbol.)
- A list of dependencies of the module (as a set of namespaced
  symbols.) Module dependencies must form a directed acyclic graph;
  circular dependencies are not allowed.
- The module constructor, as a namespaced qualified symbol that
  resolves to a *module constructor function*.

The module constructor function is a function that takes a single
argument: the module definition map. The module constructor function
must return an object implementing the `arachne.core.modules/Module`
protocol. The methods on this protocol provide access to everything a
else a module needs to be able to do.

When an application is defined, the user must specify a set of module
names to use (exact mechanism TBD.) Only the specified modules (and
their dependencies) will be considered by Arachne. In other words,
merely including a module as a maven dependency is not sufficient to
activate it and cause it to be used in an application.

## Status

Proposed

## Consequences

- Creating a basic module is lightweight, requiring only:
    - writing a short EDN file
    - writing a constructor function
    - implementing a protocol
- Consuming modules will use the same existing mechanisms as consuming
  a library.
- We will need to think of a straightforward, simple way for
  application authors to specify the modules they want to be active.
- Arachne is not responsible for any complexities of publishing,
  downloading or versioning modules
- Module versioning has all of the drawbacks of Maven's version
  system, including the pain of resolving conflicting versions. This
  situation is effectively the same as it is now with Clojure
  libraries.

