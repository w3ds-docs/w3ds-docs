---
title: "Atomicity: all or nothing"
weight: 2
---

# Atomicity: all or nothing

Atomicity means a change is either fully recorded or not recorded at all. There
is no third option where a write is half-done. This is the **A** in ACID, and
it sits directly on top of the [Write-Ahead Log](durability).

> **In plain terms**
>
> When you save something that touches several pieces of data at once, the
> eVault treats the whole save as one event. Either every piece lands together
> or none of them do, even if the power goes out halfway through. You never
> see a half-saved state, and nothing you were told was saved can later turn
> out to have only partly happened.
