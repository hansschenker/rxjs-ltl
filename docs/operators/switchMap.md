# switchMap — LTL Temporal Contract

## 1. Informal policy

`switchMap` implements the flattening policy:

```text
only latest
```

A new outer value creates a new inner stream and invalidates the previously active inner stream.

Only the currently active/latest inner may contribute downstream values.

## 2. Relevant state

A minimal model may expose:

```text
activeInner : InnerId | None
latestInner : InnerId | None
sourceCompleted : Boolean
outputCompleted : Boolean
```

Derived predicates:

```text
active(i)
latestInner(i)
noActiveInner
outputFrom(i)
```

## 3. Safety properties

### Only an active inner may emit

```text
G(outputFrom(i) -> active(i))
```

Meaning:

> Whenever a value is observed downstream from inner `i`, `i` must be active.

### Only the latest inner may emit

```text
G(outputFrom(i) -> latestInner(i))
```

Meaning:

> Whenever an inner contributes downstream, it must be the most recently selected inner.

### At most one inner is active

```text
G(activeInnerCount <= 1)
```

This differentiates `switchMap` from `mergeMap`.

## 4. Cancellation property

Once inner `i` has been replaced, no later downstream output may originate from `i`.

Conceptually:

```text
G(
  replaced(i)
  ->
  G(!outputFrom(i))
)
```

In a concrete model, the nested temporal scope may be encoded using a state proposition such as `obsolete(i)`:

```text
G(obsolete(i) -> !outputFrom(i))
```

## 5. Completion property

If the source has completed and no inner remains active, downstream completion must occur:

```text
G(
  sourceCompleted && noActiveInner
  ->
  F(outputCompleted)
)
```

If the source completes while an inner is still active, `switchMap` allows that inner to finish before downstream completion.

Therefore this would be incorrect:

```text
G(sourceCompleted -> F(outputCompleted immediately))
```

Completion depends on both source lifecycle and active-inner state.

## 6. Error property

A terminal error prevents later normal output:

```text
G(
  outputErrored
  ->
  G(!output)
)
```

A more implementation-oriented model should distinguish outer error from active-inner error and state the downstream error propagation obligations separately.

## 7. Counterexample trace

Consider:

```text
outerNext(A)
subscribe A

A.next(1)
outputFrom(A, 1)

outerNext(B)
unsubscribe A
subscribe B

A.next(2)
outputFrom(A, 2)   <-- violation
```

At the final step:

```text
outputFrom(A) = true
active(A)     = false
```

Therefore:

```text
outputFrom(A) -> active(A)
```

is false, so the behavior violates:

```text
G(outputFrom(i) -> active(i))
```

## 8. Mealy + LTL

The Mealy transition explains the local reaction:

```text
when outerNext(B)
then
  cancel current inner A
  subscribe to B
  activeInner := B
```

The LTL contract states the global observable requirement:

```text
G(outputFrom(i) -> active(i))
```

So:

```text
Mealy
  explains why A becomes obsolete

LTL
  forbids obsolete A from contributing later output

Trace
  supplies evidence of compliance or violation
```

## 9. Test obligations derived from the temporal contract

### Obligation: obsolete inner cannot emit

```text
Given inner A is active
And the outer emits B
When A emits after B has replaced it
Then no downstream value from A is observed
```

### Obligation: current inner can emit

```text
Given inner B is active
When B emits x
Then x is forwarded downstream
```

### Obligation: source completion waits for active inner

```text
Given the source completes
And inner B is still active
When B has not yet completed
Then downstream is not completed solely because the source completed
```

### Obligation: final inner completion permits downstream completion

```text
Given the source has completed
And B is the last active inner
When B completes
Then downstream eventually completes
```

## 10. Temporal fingerprint

```text
SAFETY
G(outputFrom(i) -> active(i))

LATEST
G(outputFrom(i) -> latestInner(i))

UNIQUENESS
G(activeInnerCount <= 1)

CANCELLATION
G(obsolete(i) -> !outputFrom(i))

COMPLETION
G(sourceCompleted && noActiveInner -> F(outputCompleted))
```

This compact set is the first candidate for an RxJS operator temporal contract.
