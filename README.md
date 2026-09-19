# Platform Lifecycle & Controls

### One Account State, Multiple Consequences

A customer can exist across multiple products at once.

A restriction originating in one part of that ecosystem may need to affect other products, while a restriction or participation choice belonging to one product may need to stay completely contained.

That creates a platform product question:

> **When multiple products share a customer, who owns account state, and how far should a state change propagate?**

---

## The Product Problem

A single `suspended` flag isn't enough.

Consider a fictional platform with three connected surfaces:

**Marketplace · Billing · Loyalty**

A customer can be restricted for different reasons:

- an upstream account restriction
- a loyalty-specific restriction
- their own decision to pause loyalty participation

Those states may look similar inside Loyalty, but they don't mean the same thing.

They have different **sources, authorities, scopes, effects, and recovery paths**.

The product needs to preserve those distinctions.

---

## Three Independently Owned States

Instead of treating the customer as simply `Active` or `Suspended`, maintain separate state facts.

| State | Owned by | Scope |
| --- | --- | --- |
| Upstream account restriction | Upstream authority | Marketplace + downstream Loyalty |
| Loyalty restriction | Loyalty operations | Loyalty only |
| Loyalty participation | Customer | Loyalty only |

The effective customer experience is derived from the combination of those states.

That matters because more than one can be true at the same time.

---

## The Product Decision

**State propagates according to its source, authority, and scope.**

An upstream restriction can propagate into a dependent product.

A product-specific restriction cannot reach back upstream simply because the products share the same customer.

A customer participation choice cannot alter the state of the broader account.

### Upstream account restriction

**Upstream → Marketplace + Loyalty**

Marketplace purchasing is restricted.

Loyalty earning and redemption are restricted downstream.

Billing remains available for viewing and resolving existing obligations.

Loyalty cannot independently override the upstream restriction.

### Loyalty-only restriction

**Loyalty → Loyalty only**

Loyalty earning and redemption are restricted.

Existing loyalty history and value remain visible.

Marketplace and Billing are unaffected.

### Loyalty pause / opt-out

**Customer → Loyalty participation only**

New loyalty participation, earning, and redemption stop while participation is paused.

Existing history and value are preserved.

Marketplace and Billing are unaffected.

The customer can later resume participation.

---

## Authority Matters

Defining the state isn't enough.

The product also needs to define **who has authority to change it**.

| Actor | Authority |
| --- | --- |
| Upstream authority | Apply or clear its own account restriction |
| Loyalty operator | Apply or clear a loyalty-only restriction |
| Customer | Pause or resume their own loyalty participation |

Those boundaries prevent one product or operator from changing state they don't own.

A loyalty operator cannot remove an upstream restriction.

A customer resuming loyalty participation cannot clear an administrative restriction.

An upstream restoration cannot silently erase a separate loyalty restriction or customer choice.

---

## Recovery ≠ Reset

This is where independent state becomes especially important.

Suppose a customer has:

**Upstream restriction: Restricted**  
**Loyalty restriction: Restricted**  
**Participation: Active**

The upstream issue is resolved.

Restoring the upstream account should produce:

**Upstream restriction: Clear**  
**Loyalty restriction: Restricted**  
**Participation: Active**

Marketplace access can recover.

Loyalty remains restricted.

The product does not simply set the customer back to `Active`.

> **Recovery removes the cause it resolves, not every state affecting the customer.**

The same rule applies when states overlap in a different order.

If a customer pauses Loyalty, later receives an upstream restriction, and then has that upstream restriction cleared, their original participation choice remains paused.

---

## Product Effects

Each state should be defined by what it actually changes.

| Surface | Upstream restriction | Loyalty restriction | Loyalty pause |
| --- | --- | --- | --- |
| Marketplace purchasing | Restricted | Unchanged | Unchanged |
| Billing / existing obligations | Available | Unchanged | Unchanged |
| Loyalty earning | Restricted | Restricted | Paused |
| Loyalty redemption | Restricted | Restricted | Paused |
| Loyalty history / value visibility | Available | Available | Available |
| Marketplace account state | Restricted | Unchanged | Unchanged |

This makes the lifecycle behavior explicit instead of requiring each connected product to interpret the word "suspended" independently.

---

## Try the State Explorer

→ [Launch the interactive case study](https://rtfenter.github.io/B2C-Loyalty-Product-Study/#platform)

The explorer uses one fictional customer with three independently owned state sources.

Actions include:

- Apply upstream restriction
- Apply loyalty-only restriction
- Pause loyalty participation
- Restore upstream account
- Clear loyalty-only restriction
- Resume loyalty participation

Each action exposes:

**Source → Authority → Scope → Product effects → Recovery path → Audit**

Changes are cumulative so overlapping states can be explored.

For example:

**Apply loyalty restriction → Apply upstream restriction → Restore upstream account**

Marketplace recovers while Loyalty remains restricted.

The explorer also demonstrates denied actions, such as a loyalty operator attempting to clear an upstream restriction.

The purpose isn't to operate an admin dashboard.

It's to make the boundaries behind lifecycle state visible.

---

## Auditability

State changes need to be explainable after they happen.

Each action should retain:

- timestamp
- source
- actor / authority
- reason
- affected scope
- previous state
- resulting state
- related restriction or recovery event
- outcome

That includes actions that are **denied** or produce **no change**.

A denied attempt to clear someone else's restriction is still useful product history: it explains what was attempted, why nothing changed, and which authority actually owns recovery.

---

## Rules That Need to Stay Deterministic

| Situation | Product behavior |
| --- | --- |
| Upstream restriction applied | Restrict affected upstream behavior and downstream Loyalty |
| Loyalty restriction applied | Restrict Loyalty only |
| Customer pauses Loyalty | Pause Loyalty participation only |
| Upstream restriction cleared | Remove only the upstream restriction |
| Loyalty restriction cleared | Remove only the loyalty-owned restriction |
| Customer resumes participation | Remove only the customer participation pause |
| Multiple restrictions overlap | Preserve every independently active cause |
| Unauthorized actor attempts change | Deny action without changing state |
| Duplicate state event arrives | Do not apply the same effect twice |
| Older recovery targets newer restriction | Do not clear the newer restriction |

The core rule is:

> **Each authority can change only the state it owns, and recovery removes only the cause it actually resolves.**

---

## Measurement

### Primary

**Lifecycle integrity rate**

The percentage of account-state changes that produce the intended product effects without incorrect propagation or manual correction.

### Supporting measures

- **State propagation accuracy** — did each accepted change affect only the products and capabilities within its defined scope?
- **Recovery accuracy** — did restoration remove the intended restriction without clearing independent states?
- **Manual correction rate** — how often do operators need to repair account state after a lifecycle event?
- **Lifecycle resolution time** — how long does it take an authorized recovery to reach the intended effective state?
- **Lifecycle-related support volume** — where are state changes creating customer or operator confusion?

### Guardrails

- **Unauthorized state changes** — did an actor change state outside their authority?
- **Cross-product side effects** — did a product-specific action incorrectly affect another product?
- **Lost state** — did one transition erase an independent restriction or customer choice?
- **Incorrect recovery** — did restoration return capabilities that should still be restricted?
- **Audit gaps** — can every applied, denied, and recovered state change be explained afterward?

---

## What I'd Validate

The state model can be logically correct while still creating confusing experiences for customers and operators.

I'd validate:

- Do customers understand which part of their account is restricted?
- Can they tell what remains available?
- Is the recovery path clear for each type of restriction?
- Can internal operators distinguish upstream restrictions from states they actually control?
- Are overlapping restrictions understandable without exposing unnecessary system complexity?
- Which state changes require proactive customer communication?
- Does preserving value and history while restricting actions create any misleading expectations?

The goal isn't to expose the platform's internal state model.

It's to make every restriction and recovery **scoped, predictable, explainable, and reversible by the right authority**.

---

*This is a fictionalized product study based on product patterns I've encountered professionally. Products, policies, state models, and implementation details are generalized or illustrative.*
