# RxJS-LTL Profile

This document defines a deliberately small LTL vocabulary for operator contracts.

## 1. Temporal syntax

```text
G P          always P
F P          eventually P
P U Q        P until Q

P -> Q       implication
!P           negation
P && Q       conjunction
P || Q       disjunction
```

Optional, used cautiously:

```text
X P          P in the next modeled state
```

## 2. Atomic propositions for RxJS

LTL itself does not know about Observables, subscriptions, values, timers, or schedulers.

We expose these as propositions derived from the operator model.

### Source events

```text
sourceNext(x)
sourceCompleted
sourceErrored(e)
```

### Inner-stream events

```text
innerStarted(i)
innerNext(i, x)
innerCompleted(i)
innerErrored(i, e)
```

### Downstream events

```text
output(x)
outputFrom(i)
outputCompleted
outputErrored(e)
```

### Operator state

```text
active(i)
latestInner(i)
queued(i)
pending(x)
busy
outputCount(n)
```

### Cancellation / lifecycle

```text
cancelled(i)
subscribed(i)
unsubscribed(i)
```

### Time-related propositions

```text
timerStarted(id)
timerExpired(id)
deadlineReached(id)
```

The scheduler or source supplies these time-related events. The operator reacts to them.

## 3. Temporal contract sections

Every operator should eventually be documented using the following categories where relevant.

### Safety

What must never happen?

```text
G(...)
```

### Preservation

Which eligible source events must be preserved downstream?

```text
G(inputAccepted(x) -> F(output(x)))
```

### Ordering

What relative order must be maintained?

### Concurrency

How many active inner streams may exist, and which may emit?

### Cancellation

What is forbidden after an inner or pending action is invalidated?

### Termination

Under which conditions must downstream completion or error occur?

### Liveness

What progress is guaranteed under stated assumptions?

### Assumptions

What must the environment, source, scheduler, or inner stream guarantee for liveness claims to hold?

## 4. Example temporal fingerprints

### filter

```text
Safety:
G(output(x) -> predicate(x))

Preservation:
G(sourceNext(x) && predicate(x) -> F(output(x)))
```

### take(n)

```text
Cardinality:
G(outputCount <= n)

Termination:
G(outputCount = n -> F(outputCompleted))

Post-termination:
G(outputCompleted -> G(!output))
```

### switchMap

```text
Current-inner safety:
G(outputFrom(i) -> active(i))

Latest-inner safety:
G(outputFrom(i) -> latestInner(i))

Uniqueness:
G(activeInnerCount <= 1)

Completion:
G(sourceCompleted && noActiveInner -> F(outputCompleted))
```

### concatMap

```text
Serial execution:
G(activeInnerCount <= 1)

Conditional queue progress:
G(
  queued(i) && predecessorsEventuallyTerminate(i)
  ->
  F(active(i))
)
```

### exhaustMap

```text
Single active inner:
G(activeInnerCount <= 1)

Ignore while busy:
G(
  busy && sourceNext(x)
  ->
  !startInnerFor(x)
)
```

### mergeMap

```text
Valid-inner output:
G(outputFrom(i) -> active(i))
```

No uniqueness property is imposed: overlap is permitted.

## 5. Quantitative time

Plain LTL expresses temporal ordering, not physical durations such as 300 ms.

For time-sensitive RxJS operators we can expose timer events into the state/event model:

```text
timerExpired(id)
```

Then a temporal contract can reason about the result:

```text
G(
  pending(x) && timerExpired(timerFor(x))
  ->
  F(output(x))
)
```

If precise metric-time properties become central, this project should separately evaluate Metric Temporal Logic or timed-automata approaches rather than overloading ordinary LTL.

## 6. Translation to test obligations

One aim of this repository is:

```text
Plain-language requirement
        ↓
LTL property
        ↓
Test obligation
```

Example:

Requirement:

```text
An obsolete switchMap inner must never emit downstream.
```

LTL:

```text
G(outputFrom(i) -> active(i))
```

Test obligation:

```text
Given inner A is active
And the outer source emits B
When A emits after cancellation
Then no downstream value from A is observed
```

This bridge is expected to be one of the strongest candidates for later integration into `rxjs-formal-way`.
