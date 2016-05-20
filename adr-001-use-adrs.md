# Architecture Decision Record: Use ADRs

## Context

Arachne has several very explicit goals that make the practice and
discipline of architecture very important:

- We want to think deeply about all our architectural decisions,
  exploring all alternatives and making a careful, considered,
  well-researched choice.
- We want to be as transparent as possible in our decision-making
  process.
- We don't want decisions to be made unilaterally in a
  vacuum. Specifically, we want to give our steering group the
  opportunity to review every major decision.
- Despite being a geographically and temporally distributed team, we
  want our contributors to have a strong shared understanding of the
  technical rationale behind decisions.
- We want to be able to revisit prior decisions to determine fairly if
  they still make sense, and if the motivating circumstances or
  conditions have changed.

## Decision

We will document every architecture-level decision for Arachne and its
core modules with an
[Architecture Decision Record](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions). These
are a well structured, relatively lightweight way to capture
architectural proposals. They can serve as an artifact for discussion,
and remain as an enduring record of the context and motivation of past
decisions.

The workflow will be:

1. A developer creates an ADR document outlining an approach for a
   particular question or problem. The ADR has an initial status of "proposed."
2. The developers and steering group discuss the ADR. During this
   period, the ADR should be updated to reflect additional context,
   concerns raised, and proposed changes.
3. Once consensus is reached, ADR can be transitioned to either an
   "accepted" or "rejected" state.
4. Only after an ADR is accepted should implementing code be committed
   to the master branch of the relevant project/module.
5. If a decision is revisited and a different conclusion is reached, a
   new ADR should be created documenting the context and rationale for
   the change. The new ADR should reference the old one, and once the
   new one is accepted, the old one should (in its "status" section)
   be updated to point to the new one. The old ADR should not be
   removed or otherwise modified except for the annotation pointing to
   the new ADR.

## Status

Accepted

## Consequences

1. Developers must write an ADR and submit it for review before
   selecting an approach to any architectural decision -- that is, any
   decision that affects the way Arachne or an Arachne application is
   put together at a high level.
2. We will have a concrete artifact around which to focus discussion,
   before finalizing decisions.
3. If we follow the process, decisions will be made deliberately, as a group.
4. The master branch of our repositories will reflect the high-level
   consensus of the steering group.
5. We will have a useful persistent record of why the system is the way it is.
