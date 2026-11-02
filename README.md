# RxJS + Linear Temporal Logic

A study repository for applying **Linear Temporal Logic (LTL)** to the formal description of **RxJS operator behavior**.

The goal is not to replace the existing Mealy-machine analysis of RxJS operators. The goal is to add a complementary layer that describes what must hold across an entire execution trace.

## Core idea

An RxJS operator can be studied at several levels:

```text
Cω notation
  What values/cardinalities can flow?

Mealy machine
  What happens when a particular event occurs?

LTL
  What must always or eventually hold across the behavior?

TLA+
  Does the complete behavioral specification satisfy those properties?

Trace
  What behavior actually occurred?
```

The key distinction is:

> **Mealy describes transitions. LTL describes behaviors.**

If an operator permits a set of behaviors `Behaviors(Op)`, and `φ` is its temporal contract, operator correctness can be stated as:

```text
for every behavior b in Behaviors(Op):
  b satisfies φ
```

or formally:

```text
Op ⊨ φ
```

## Why LTL is useful for RxJS

RxJS executions are naturally linear traces:

```text
s0 -> s1 -> s2 -> s3 -> ...
```

LTL is designed to express properties over exactly such traces.

Typical RxJS requirements map naturally to temporal properties:

- **Safety** — something bad must never happen.
- **Liveness** — something required must eventually happen.
- **Ordering** — one event must precede or constrain another.
- **Cancellation** — after invalidation, certain outputs must never occur.
- **Termination** — completion/error conditions constrain future behavior.
- **Concurrency policy** — active inner streams obey operator-specific rules.

## Minimal RxJS-LTL profile

This repository initially emphasizes a small, teachable subset:

```text
G P        always P
F P        eventually P
P U Q      P holds until Q
P -> Q     implication
!P         not P
P && Q     conjunction
P || Q     disjunction
```

The `X` (next) operator is treated cautiously because it is not stuttering-invariant.

## Example: switchMap

The defining safety property can be written as:

```text
G(outputFrom(i) -> active(i))
```

Meaning:

> Always, if an inner stream contributes a downstream value, that inner stream must be the currently active inner.

A stronger latest-inner formulation is:

```text
G(outputFrom(i) -> i = latestInner)
```

This compactly captures the observable switching policy.

## Repository structure

```text
docs/
  ltl-foundations.md
  rxjs-ltl-profile.md
  roadmap.md
  operators/
    switchMap.md
```

Future work will add temporal contracts for representative operator families and determine which parts should later be incorporated into `rxjs-formal-way`.

## Relationship to rxjs-formal-way

This repository is deliberately a research/study space.

When the LTL approach is mature enough, the parts that materially strengthen the formal operator workflow can be transferred into:

```text
hansschenker/rxjs-formal-way
```

The intended integration point is likely:

```text
Cω
  ↓
Mealy
  ↓
BDD
  ↓
STT
  ↓
Invariants
  ↓
LTL Temporal Contract
  ↓
Test Obligations
  ↓
TLA+
  ↓
Trace
  ↓
Plain-language documentation
```

## Contributors

- Hans Schenker — project direction, RxJS formalization framework, operator semantics
- **GPT-5.6 Sol (OpenAI)** — main contributor to the initial LTL analysis, temporal-contract model, and repository structure
