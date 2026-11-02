# Roadmap

The repository should remain a focused study of **LTL as a specification language for RxJS operator behavior**.

## Phase 1 — Foundations

- [x] Define behavior as a sequence of states
- [x] Define safety versus liveness
- [x] Establish a minimal RxJS-LTL syntax profile
- [x] Define RxJS-specific atomic propositions
- [x] Explain the Mealy/LTL division of responsibility
- [x] Add the first worked operator: `switchMap`

## Phase 2 — Representative operator families

Study operators chosen to expose different temporal concerns.

### Filtering / cardinality

- [ ] `filter`
- [ ] `take`
- [ ] `first`
- [ ] `distinctUntilChanged`

### Flattening policies

- [ ] `mergeMap`
- [ ] `concatMap`
- [x] `switchMap`
- [ ] `exhaustMap`

### Time-sensitive operators

- [ ] `debounceTime`
- [ ] `auditTime`
- [ ] `throttleTime`
- [ ] `delay`
- [ ] `timeout`

### Combination / joining

- [ ] `combineLatest`
- [ ] `zip`
- [ ] `withLatestFrom`

### Aggregation / state

- [ ] `scan`
- [ ] `reduce`
- [ ] `buffer`
- [ ] `window`

### Sharing / multicasting

- [ ] `share`
- [ ] `shareReplay`

## Phase 3 — Formal questions

- [ ] Define event/state trace semantics precisely
- [ ] Decide which properties should be state predicates versus event predicates
- [ ] Define finite-trace treatment for completed RxJS subscriptions
- [ ] Study stuttering invariance systematically
- [ ] Determine exact use policy for `X`
- [ ] Define fairness assumptions for scheduler/source/inner progress
- [ ] Separate operator guarantees from environment assumptions
- [ ] Define quantitative-time boundary of ordinary LTL
- [ ] Evaluate whether Metric Temporal Logic adds practical value

## Phase 4 — Translation pipeline

Explore:

```text
Plain-language rule
  ↓
LTL
  ↓
test obligation
```

and:

```text
Mealy model
  ↓
generated propositions
  ↓
LTL temporal contract
  ↓
trace checker / model checker
```

## Phase 5 — Relationship to TLA+

Study how the RxJS-LTL profile maps into the existing `rxjs-tla-plus` and `rxjs-formal-way` experiments.

Questions:

- Which LTL properties become TLA+ temporal formulas directly?
- Which safety properties are better represented as invariants?
- Which liveness properties require fairness?
- Can a Mealy operator model generate both transition rules and temporal properties?
- Can counterexample traces be translated back into RxJS four-lane diagrams?

## Phase 6 — Extraction into rxjs-formal-way

Do not integrate prematurely.

Only transfer concepts that demonstrably improve the formal workflow.

Likely candidates:

1. **LTL Temporal Contract** section on every operator page
2. Safety/liveness classification
3. Assumption/guarantee separation
4. Temporal-contract-to-test-obligation translation
5. Stuttering-aware guidance
6. TLA+ temporal-property mapping

Proposed eventual workflow:

```text
Cω
  ↓
Mealy 6-tuple
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

## Definition of "complete enough to integrate"

`rxjs-ltl` should be considered mature enough for extraction only after:

- representative first-order, higher-order, time-sensitive, lifecycle, and sharing operators have been modeled;
- the event/state trace model is stable;
- safety and liveness conventions are consistent;
- assumptions and fairness are explicit;
- generated test obligations have been demonstrated on multiple operators;
- the relationship with TLA+ is documented;
- at least one automated checking experiment exists.
