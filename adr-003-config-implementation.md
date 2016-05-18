# Architecture Decision Record: Datomic-based Configuration

## Context

[ADR-002](adr-002-configuration.md) indicates that we will store the
entire application config in a single rich data structure with a schema.

This implies that it should be possible to easily search, query and
update the configuration value. It also implies that the configuration
value is general enough to store arbitrary data; we don't know what
kinds of things users or module authors will need to include.

Systems that allow you to search, query and update arbitrary schema'd
data are called databases. We should look at in-memory databases for
implementations of the configuration.

The selected database needs to have an *extensible* schema
system. Because each module will contribute its own portion of the
schema, it needs to be possible to iteratively expand the schema
rather than declaring it all at once.

Ideally, the selected database should be easy to use with Clojure, and
be familiar enough to a critical mass of module developers that they
can get started with it right away.

There are two classes of systems that roughly meet this criteria:

- Datomic
- RDF with a RDFs/OWL schema

In order to make it usable by the largest number of people, Arachne's
core needs to function entirely on permissive open-source software; a
hard requirement for proprietary or GPL-licensed code will make it
unviable for many users.

## Decision

We will provide a facade API that lets each user use Datomic (free or
pro) or DataScript, according to their preference. Those with an
existing investment in Datomic will likely prefer that, while those
that have an open-source requirement will probably prefer DataScript.

The facade will present a functional, value-based interface for
Datalog queries, pull expressions and updates (via Datomic-style
txdata). It will expose databases as values, not as stateful entities,
to emphasize the intended uses of the configuration.

Arachne will also provide an implementation that multiplexes all operations
to both Datomic *and* DataScript, throwing an exception if the results
don't match exactly. This is intended for use by module authors, to
ensure that their modules work with both possible implementations.

## Status

Proposed

## Consequences

- The configuration will be able to store any kind of data Datomic can store.
- The configuration will have an extensible schema.
- The capabilities of the configuration system will be well documented.
- Arachne and Arachne modules can optionally define meta-attributes
  (attributes on attributes) to increase the expressivity of the
  schema used by modules.
- Module authors will need be comfortable with Datomic.
- Module authors will need to rely upon the subset of features that
  are fully supported in the same way by both Datomic and DataScript.

