# Architecture Decision Record: Configuration Updates

## Context

A core part of the process of developing an application is making changes to its configuration. With its emphasis on configuration, this is even more true of Arachne than with most other web frameworks.

In a development context, developers will want to see these changes reflected in their running application as quickly as possible. Keeping the test/modify cycle short is an important goal.

However, accommodating change is a source of complexity. Extra code would be required to handle  "update" scenarios. Components are initialized with a particular configuration in hand. While it would be possible to require that every component support an `update` operation to receive an arbitrary new config, implementing this is non-trivial and would likely need to involve conditional logic to determine the ways in which the new configuration is different from the old. If any mistakes where made in the implementation of `update`, *for any component*, such that the result was not identical to a clean restart, it would be possible to put the system in an inconsistent, unreproducible state.

The "simplest" approach is to avoid the issue and completely discard and rebuild the Arachne runtime ([ADR-006](adr-006-core-runtime)) every time the configuration is updated. Every modification to the config would be applied via a clean start, guaranteeing reproducibility and a single code path.

However, this simple baseline approach has two major drawbacks:

1. The shutdown, initialization, and startup times of the entire set of components will be incurred every time the configuration is updated.
2. The developer will lose any application state stored in the components whenever the config is modified.

The startup and shutdown time issues are potentially problematic because of the general increase to cycle time. However, it might not be too bad depending on exactly how long it takes sub-components to start. Most commonly-used components take only a few milliseconds to rebuild and restart. This is a cost that most Component workflows absorb without too much trouble.

The second issue is more problematic. Not only is losing state a drain on overall cycle speed, it is a direct source of frustration, causing developers to repeat the same tasks over and over. It will mean that touching the configuration has a real cost, and will cause developers to be hesitant to do so.


### Prior Art

There is a library designed to solve the startup/shutdown problem, in conjunction with Component: [Suspendable](https://github.com/weavejester/suspendable). It is not an ideal fit for Arachne, since it focuses on suspending and resuming the same Component instances rather than rebuilding, but its approach may be instructive.

## Decision

Whenever the configuration changes, we will use the simple approach of stopping and discarding the entire old Arachne runtime (and all its components), and starting a new one.

To mitigate the issue of lost state, Arachne will provide a new protocol called `Preservable` (name subject to change, pending a better one.) Components may optionally implement `Preservable`; it is not required. `Preservable` defines a single method, `preserve`. 

Whenever the configuration changes, the following procedure will be used:

1. Call `stop` on the old runtime.
2. Instantiate the new runtime.
3. For all components in the new runtime which implement `Preservable`, invoke the `preserve` function, passing it the corresponding component from the old runtime (if there is one).
4. The `preserve` function will selectively copy state out of the old, stopped component into the new, not-yet-started component. It should be careful not to copy any state that would be invalidated by a configuration change.
5. Call `start` on the new runtime.

Arachne will not provide a mitigation for avoiding the cost of stopping and starting individual components. If this becomes a pain point, we can explore solutions such as that offered by Suspendable.
 
## Status

PROPOSED

## Consequences

- The basic model for handling changes to the config will be easy to implement and reason about.
- It will be possible to develop with stateful components without losing state after a configuration change.
- Only components which need preservable state need to worry about it.
- The default behavior will prioritize correctness.
- It is _possible_ to write a bad `preserve` method which copies elements of the old configuration.
- However, because all copies are explicit, it should be easy to avoid writing bad `preserve` methods.

