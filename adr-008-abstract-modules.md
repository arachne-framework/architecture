# Architecture Decision Record: Abstract Modules

## Context

One design goal of Arachne is to have modules be relatively easily swappable. Users should not be permanently committed to particular technical choices, but instead should have some flexibility in choosing their preferred tech, as long as it exists in the form of an Arachne module.

Some examples of the alternative implementations that people might wish to use for various parts of their application:

- HTTP Server: Pedestal or Ring
- Database: Datomic, an RDBMS or one of many NoSQL options.
- HTML Templating: Hiccup, Enlive, StringTemplate, etc.
- Client-side code: ClojureScript, CoffeeScript, Elm, etc.
- Authentication: Password-based, OpenID, Facebook, Google, etc.
- Emailing: SMTP, one of many third-party services.

This is only a representative sample; the actual list is unbounded.

The need for this kind of flexibility raises some design concerns: 

**Capability**. Users should always be able to leverage the full power of their chosen technology. That is, they should not have to code to the "least common denominator" of capability. If they use Datomic Pro, for example, they should be able to write Datalog and fully utilize the in-process Peer model, not be restricted to an anemic "ORM" that is also compatible with RDBMSs.

**Uniformity**. At tension with capability is the desire for uniformity; where the feature set of two alternatives is *not* particularly distinct, it is desirable to use a common API, so that implementations can be swapped out with little or no effort. For example, the user-facing API for sending a single email should (probably) not care whether it is ultimately sent via a local Sendmail server or a third-party service.

**Composition**. Modules should also *compose* as much as possible, and they should be as general as possible in their dependencies to maximize the number of compatible modules. In this situation, it is actually desirable to have a "least common denominator" that modules can have a dependency on, rather than depending on specific implementations. For example, many modules will need to persist data and ultimately will need to work in projects that use Datomic or SQL. Rather than providing multiple versions, one for Datomic users and another for SQL, it would be ideal if they could code against a common persistence abstraction, and therefore be usable in *any* project with a persistence layer.

### What does it mean to use a module?

The following list enumerates the ways in which it is possible to "use" a module, either from a user application or from another module. (See [ADR-004](ADR-004-module-loading.md)).

1. You can call code that the module provides (the same as any Clojure library.)
2. You can extend a protocol that the module provides (the same as any Clojure library.)
3. You can read the attributes defined in the module from the configuration.
4. You can write configuration data using the attributes defined in the module.

These tools allow the definition of modules with many different kinds of relationships to each other. Speaking loosely, these relationships can correspond to other well-known patterns in software development including composition, mixins, interface/implementation, inheritance, etc.

## Decision

In order to simultaneously meet the needs for capability, uniformity and composition, Arachne's core modules will (as appropriate) use the pattern of *abstract modules*.

Abstract modules define certain attributes (and possibly also corresponding init script DSLs) that describe entities in a particular domain, *without* providing any runtime implementation which uses them. Then, other modules can "implement" the abstract module, reading the abstract entities and doing something concrete with them at runtime, as well as defining their own more specific attributes.

In this way, user applications and dependent modules can rely either on the common, abstract module or the specific, concrete module as appropriate. Coding against the abstract module will yield a more generic "least common denominator" experience, while coding against a specific implementor will give more access to the unique distinguishing features of that particular technology, at the cost of generality.

Similar relationships should hold in the library code which modules expose (if any.) An abstract module, for example, would be free to define a protocol, intended to be implemented concretely by code in an implementing module.

This pattern is fully extensible; it isn't limited to a single level of abstraction. An abstract module could itself be a narrowing or refinement of another, even more general abstract module.

### Concrete Example

As mentioned above, Arachne would like to support both Ring and Pedestal as HTTP servers. Both systems have a number of things in common:

- The concept of a "server" running on a port.
- The concept of a URL path/route
- The concept of a terminal "handler" function which receives a request and returns a response.

They also have some key differences:

- Ring composes "middleware" functions, whereas Pedestal uses "interceptor" objects
- Asynchronous responses are handled differently

Therefore, it makes sense to define an abstract HTTP module which defines the basic domain concepts; servers, routes, handlers, etc. Many dependent modules and applications will be able to make real use of this subset.

Then, there will be the two modules which provide concrete implementations; one for Pedestal, one for Ring. These will contain the code that actually reads the configuration, and at runtime builds appropriate routing tables, starts server instances, etc. Applications which wish to make direct use of a specific feature like Pedestal interceptors may freely do so, using attributes defined by the Pedestal module.

## Status

PROPOSED

## Consequences

- If modules or users want to program against a "lowest common denominator" abstraction, they may do so, at the cost of the ability to use the full feature set of a library.
- If modules or users want to use the full feature set of a library, they may do so, at the cost of being able to transparently replace it with something else.
- There will be a larger number of different Arachne modules available, and their relationships will be more complex.
- Careful thought and architecture will need to go into the factoring of modules, to determine what the correct general elements are.