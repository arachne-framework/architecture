# Architecture Decision Record: User Facing Configuration

## Context

Per [ADR-003](adr-003-config-implementation.md), Arachne uses
Datomic-shaped data for configuration. Although this is a flexible,
extensible data structure which is a great fit for programmatic
manipulation, in its literal form it is quite verbose.

It is quite difficult to understand the structure of Datomic data by
reading its native textual representation, and it is similarly hard to
write, containing enough repeated elements that copying and pasting
quickly becomes the default.

One of Arachne's core values is ease of use and a fluent experience
for developers. Since much of a developer's interaction with Arachne
will be writing to the config, it is of paramount importance that
there be some easy way to create configuration data.

The question is, what is the best way for developers of Arachne
applications to interact with their application's configuration?

#### Option: Raw Datomic Txdata

This would require end users to write Datomic transaction data by hand
in order to configure their application.

This is the "simplest" option, and has the fewest moving
parts. However, as mentioned above, it is very far from ideal for
human interactions.

#### Option: Custom EDN data formats

In this scenario, users would write EDN data in some some nested
structure of maps, sets, seqs and primitives. This is currently the
most common way to configure Clojure applications.

Each module would then need to provide a mapping from the EDN config
format to the underlying Datomic-style config data.

Because Arachne's configuration is so much broader, and defines so
much more of an application than a typical application config file, 
it is questionable if standard nested EDN data would be a good fit 
for representing it.

#### Option: Code-based configuration

Another option would be to go in the direction of some other
frameworks, such as Ruby on Rails, and have the user-facing
configuration be *code* rather than data.

It should be noted that the primary motivation for having a
data-oriented configuration language, that it makes it easier to
interact with programmatically, doesn't really apply in Arachne's
case. Since applications are always free to interact richly with
Arachne's full configuration database, the ability to programmatically
manipulate the precursor data is moot. As such, one major argument
against a code-based configuration strategy does not apply.

## Decision

Developers will have the option of writing configuration using either
native Datomic-style, data, or code-based *configuration
scripts*. Configuration scripts are Clojure files which, when
evaluated, update a configuration stored in an atom currently in
context (using a dynamically bound var.)

Configuration scripts are Clojure source files in a distinct directory
that by convention is *outside* the application's classpath:
configuration code is conceptually and physically separate from
application code. Conceptually, loading the configuration scripts
could take place in an entirely different process from the primary
application, serializing the resulting config before handing it to the
runtime application.

To further emphasize the difference between configuration scripts and
runtime code, and because they are not on the classpath, configuration
scripts will not have namespaces and will instead include each other
via Clojure's `load` function.

Arachne will provide code supporting the ability of module authors to
write "configuration DSLs" for users to invoke from their
configuration scripts. These DSLs will emphasize making it easy to
create appropriate entities in the configuration. In general, DSL
forms will have an imperative style: they will convert their arguments
to configuration data and immediately transact it to the context
configuration.

As a trivial example, instead of writing the verbose configuration data:

```clojure
{:arachne/id :my.app/server
 :arachne.http.server/port 8080
 :arachne.http.server/debug true}
 ```

You could write the corresponding DSL:

```clojure
(server :id :my.app/server, :port 8080, :debug true)
```

Note that this is an illustrative example and does not represent the
actual DSL or config for the HTTP module.

DSLs should make heavy use of Spec to make errors as comprehensible as possible.

## Status

Proposed

## Consequences

- It will be possible for end users to define their configuration
  without writing config data by hand.
- Users will have access to the full power of the Clojure programming
  language when configuring their application. This grants a great deal
  of power and flexibility, but also the risk of users doing inadvisable
  things in their config scripts (e.g, non-repeatable side effects.)
- Module authors will bear the responsibility of providing
  an appropriate, user-friendly DSL interface to their configuration data.
- DSLs can compose; any module can reference and re-use the DSL
  definitions included in modules upon which it depends.

