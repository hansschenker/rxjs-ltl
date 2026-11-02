# LTL Foundations for RxJS

## 1. Behavior as a sequence of states

For this repository, an RxJS execution is viewed as a behavior:

```text
b = s0, s1, s2, s3, ...
```

Each state records the information relevant to the operator being studied, for example:

```text
activeInner
outerCompleted
pendingValue
outputCount
errored
completed
```

LTL formulas are interpreted over such behaviors.

## 2. Core temporal operators

### Globally / Always

```text
G(P)
```

P must hold at every relevant position in the behavior.

RxJS use:

```text
G(outputCount <= 3)
```

for the cardinality restriction of `take(3)`.

### Eventually

```text
F(P)
```

P must hold at some point in the future.

RxJS use:

```text
G(sourceCompleted && noActiveInner -> F(outputCompleted))
```

### Until

```text
P U Q
```

P must continue to hold until Q becomes true.

This is useful for pending states, active inner lifetimes, and waiting policies.

## 3. Safety and liveness

### Safety

A safety property says that a bad state or bad event pattern must never occur.

Typical form:

```text
G(P)
```

Examples:

```text
switchMap:
G(outputFrom(i) -> active(i))

take(n):
G(outputCount <= n)

filter(predicate):
G(output(x) -> predicate(x))
```

A safety violation can normally be exposed by a finite counterexample trace.

### Liveness

A liveness property says that some required progress eventually occurs.

Typical form:

```text
G(P -> F(Q))
```

Examples:

```text
sourceCompleted && noActiveInner
  -> eventually outputCompleted

queued(i) && allPredecessorsEventuallyTerminate(i)
  -> eventually active(i)
```

Liveness properties require care because they often depend on environment assumptions.

## 4. Assumptions and guarantees

RxJS operators cannot guarantee progress that depends on an external source terminating.

For example:

```ts
source$.pipe(
  concatMap(() => NEVER)
)
```

A queued outer value cannot become active while the current inner never completes.

Therefore this is too strong:

```text
G(queued(i) -> F(active(i)))
```

A better contract is conditional:

```text
Assumption:
  preceding active inners eventually terminate

Guarantee:
  queued inner i eventually becomes active
```

This suggests an assume/guarantee form:

```text
EnvironmentAssumptions -> OperatorGuarantees
```

## 5. Why LTL complements Mealy machines

A Mealy machine describes local reaction:

```text
(state, event)
  ->
(nextState, actions)
```

LTL describes global constraints over the resulting behavior:

```text
G(P -> F(Q))
```

For RxJS:

```text
Mealy
  explains how an operator reacts

LTL
  states what all valid executions must satisfy
```

The two layers are complementary rather than competing.

## 6. Why X (Next) is treated cautiously

The LTL next operator:

```text
X(P)
```

means that P holds in the immediately following state.

This is often convenient, but it is sensitive to stuttering: inserting an observationally unchanged state can alter the truth of an `X` formula.

Because the broader RxJS formalization is influenced by Lamport-style behavior semantics and TLA+, this repository prefers stuttering-tolerant formulations whenever practical.

Primary operators:

```text
G
F
U
```

Use `X` only when the model's notion of an immediate next state is intentionally part of the specification.

## 7. Operator correctness

Let:

```text
Behaviors(Op)
```

be all behaviors allowed by an operator model.

Let:

```text
TemporalContract(Op)
```

be the conjunction of its LTL properties.

Then:

```text
Op is correct
iff
every b in Behaviors(Op) satisfies TemporalContract(Op)
```

Formally:

```text
Op ⊨ TemporalContract(Op)
```

This is the central correctness idea explored in this repository.
