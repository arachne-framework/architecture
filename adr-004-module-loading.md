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
process. Additionally, since modules can depend upon each other, they
must specify which modules they depend upon.

Ideally there will be as little overhead as possible for creating and
consuming modules.

Some of the general problems associated with plugin/module systems include:

- Finding and downloading the implementation of the module.
- Discovering and activating the correct set of installed modules.
- Managing module versions and dependencies.

There are some existing systems for modularity in the Java
ecosystem. The most notable is OSGi, which provides not only a module
system addressing the concerns above, but also service runtime with
classpath isolation, dynamic loading and unloading and lazy
activation.

OSGi (and other systems of comparable scope) are overkill for
Arachne. Although they come with benefits, they are very heavyweight
and carry a high complexity burden, not just for Arachne development
but also for end users. Specifically, Arachne applications will be
drastically simpler if (at runtime) they exist as a straightforward
codebase in a single classloader space. Features like lazy loading and
dynamic start-stop are likewise out of scope; the goal is for an
Arachne runtime itself to be lightweight enough that starting and
stopping when modules change is not an issue.

## Decision

Arachne will not be responsible for packaging, distribution or
downloading of modules. These jobs will be delegated to an external
dependency management & packaging tool. Initially, that tool will be
Maven/Leiningen/Boot, or some other tool that works with Maven
artifact repositories, since that is currently the standard for JVM
projects.

Modules that have a dependency on another module must specify a
dependency using Maven (or other dependency management tool.)

Arachne will provide no versioning system beyond what the packaging
tool provides.

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
argument: the module definition map (exactly as read, containing
not-yet-resolved symbols.) The module constructor function must return
an object implementing the `arachne.core.modules/Module` protocol. The
methods on this protocol provide access to everything a else a module
needs to be able to do.

The module constructor function should avoid doing anything related to
the logic of the application or module beyond returnign a Module
instance. The module object should also avoid storing any state: the
module object's methods should be pure functions of their arguments
and the initialized module.

When an application is defined, the user must specify a set of module
names to use (exact mechanism TBD.) Only the specified modules (and
their dependencies) will be considered by Arachne. In other words,
merely including a module as a dependency in the package manager is
not sufficient to activate it and cause it to be used in an
application.

Similarly, in a deployment context, Arachne will be oriented towards
"immutable" or blue/green deployments, avoiding downtime at the
infrastructure level rather than attempting to support hot reloading
in a single process. This substantially simplifies the design of
Arachne in general, particularly the module system.

## Status

Proposed

## Consequences

- Creating a basic module is lightweight, requiring only:
    - writing a short EDN file
    - writing a constructor function
    - implementing a protocol
- Consuming modules will use the same existing mechanisms as consuming
  a library.
- Arachne is not responsible for getting code on the classpath; that
  is a separate and prior concern.
- We will need to think of a straightforward, simple way for
  application authors to specify the modules they want to be active.
- Arachne is not responsible for any complexities of publishing,
  downloading or versioning modules
- Module versioning has all of the drawbacks of Maven's version
  system, including the pain of resolving conflicting versions. This
  situation is effectively the same as it is now with Clojure
  libraries.
- A single Maven artifact can contain several Arachne modules (whether
  this is ever desirable is another question.)

