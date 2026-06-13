---
title: "Consistency: invariants hold"
weight: 4
---

# Consistency: invariants hold

Consistency means the eVault only ever moves from one valid state to another.
A change that would break a rule is refused; a change that is accepted is one
the rules already permit. This is the **C** in ACID.

> **In plain terms**
>
> The eVault has a short list of rules every piece of data has to satisfy:
> the right shape, the right signatures, sensible permissions. Every change is
> checked against those rules before it is allowed in. Because all three
> replicas check the same rules in the same order, they never disagree about
> what is in the store.
