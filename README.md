# DefinitiveSpec (DSpec)

> **A Constraint Satisfaction Engine for AI-Assisted Software Development**
> *Spec-first. Evidence-based. Verifiable by construction.*

---

## What Is DefinitiveSpec?

DefinitiveSpec is a structured specification methodology and AI agent protocol that treats **every design decision as a constraint satisfaction problem**. Instead of asking an LLM to write code, you use DefinitiveSpec to produce machine-readable, formally-grounded specification artifacts — *DSpec files* — that serve as a single source of truth from which code, tests, and documentation can be reliably derived.

The spec-driven development movement has a thesis: intent should precede code. That thesis is correct. But most tools built around it share a hidden assumption that undermines their own premise — that specifications are documents for humans, and the AI's job is to read them and proceed. DefinitiveSpec inverts this. Specifications are **typed, schema-validated contracts**. The AI is a constraint satisfaction engine operating under a 9-axiom kernel with mandatory self-critique at every generation step. And the reasoning chain itself — not just the output — is a first-class, auditable artifact.

The practical difference: in a DSpec session, the AI cannot silently assume a business rule, cannot reference an unverified library, cannot emit an artifact that contradicts a domain invariant, and cannot proceed past a genuine blocker by guessing. These are not guidelines. They are enforced halts.

```
Requirement → Model → Design → API → Code Spec → Test Spec → Implementation
                ↑                              ↓
        Single source of truth      Machine-generated from contracts
```

---

## Why This Matters: The Research Foundation

DefinitiveSpec is not a prompt template. It is a principled application of four decades of formal methods research to the practical problem of LLM-assisted development.

### 1. Specification as the Missing Link

A 2024 paper from CMU and Stanford, *"Specifications: The Missing Link to Making the Development of LLM Systems an Engineering Discipline"*, argues that the fundamental constraint on LLM reliability is the absence of precise task specifications. The paper observes that nearly every category of LLM failure — hallucinations, boundary violations, unsafe outputs — traces back to underspecification: the model was never told *precisely* what it must and must not do. DefinitiveSpec operationalizes this insight by requiring formal `contracts` (preconditions, postconditions, invariants, error catalog entries) before any code artifact is considered valid.

### 2. Hoare Logic: Correctness by Contract

C.A.R. Hoare's 1969 paper *"An Axiomatic Basis for Computer Programming"* (CACM, Vol. 12, No. 10) introduced the **Hoare triple** `{P} S {Q}`: if precondition *P* holds before statement *S* executes and *S* terminates, postcondition *Q* holds after. This mathematical framework is the direct intellectual ancestor of DefinitiveSpec's `contracts` block:

```yaml
contracts:
  pre:
    - "orderId is a valid non-null UUID"
    - "requesterId has role ADMIN or is order.ownerId"
  post:
    - "order.status === 'DELETED'"
    - "order.version is incremented by 1"
  invariants:
    - "No SQL DELETE issued against Order table"
```

Every `VERIFY` keyword in a `logic_blueprint` is a direct encoding of a Hoare assertion. Every `TRANSACTIONAL_BLOCK` enforces atomicity semantics that correspond to the composition rule in Hoare's calculus. The `throws_errors` list corresponds to the partial-correctness carve-out: if the program does not terminate normally, the named error defines what the caller can observe.

**Key reference:** Hoare, C.A.R. (1969). *An Axiomatic Basis for Computer Programming.* Communications of the ACM, 12(10), 576–580.

### 3. Domain-Driven Design: Models That Reflect Reality

Eric Evans' *Domain-Driven Design: Tackling Complexity in the Heart of Software* (Addison-Wesley, 2003) established that the biggest source of accidental complexity in large systems is the gap between the business domain model and the software implementation. DDD introduced the patterns DefinitiveSpec uses as first-class citizens: **Aggregate Roots**, **Entities**, **Value Objects**, **Bounded Contexts**, and **Ubiquitous Language**.

DefinitiveSpec enforces DDD boundaries through its `DOMAIN_DRIVEN_BOUNDARIES` axiom:

- An `ENTITY` with `part_of: model.Order` may never be `PERSIST`-ed directly. You must route mutations through the `AGGREGATE_ROOT`.
- The `AggregateViolation` guard fires at design time — before a single line of code is written.
- `model.stereotypes` map precisely to Evans' taxonomy: `AGGREGATE_ROOT | ENTITY | VALUE_OBJECT | DTO`.

This is not decorative classification. It is a structural constraint that prevents the most common class of domain integrity bugs in event-driven and distributed systems.

**Key reference:** Evans, E. (2003). *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Addison-Wesley. ISBN 0-321-12521-5.

### 4. Property-Based Testing: Invariants as Executable Truth

Claessen and Hughes introduced QuickCheck in their 2000 ICFP paper *"QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs"*, establishing the paradigm of **property-based testing (PBT)**: instead of writing individual test cases, you write properties that must hold for *all* valid inputs, and let the framework generate adversarial counter-examples automatically.

DefinitiveSpec integrates PBT as a first-class output artifact. When a handler's `contracts.invariants` are non-empty and the model defines numeric ranges, enum domains, or collection sizes, DefinitiveSpec automatically generates a `test.type: property_based` spec alongside the integration test:

```yaml
test:
  CreateOrderPropertyTest:
    type: property_based
    target: code.CreateOrderHandler
    invariants_under_test:
      - "Order.total >= 0"          # verbatim from contracts.invariants
    generators:
      dto.total:
        strategy: range
        min: 0
        max: 1_000_000
        edge_cases: [0, 0.001, 999_999.999]
    shrinking: true
    runs: 100
```

When a counterexample is found, it becomes a permanent regression case. The generator is never narrowed to hide the failure — that would be a `SPEC_FIRST` violation.

**Key reference:** Claessen, K. & Hughes, J. (2000). *QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs.* Proceedings of ICFP 2000, ACM SIGPLAN, 268–279. DOI: 10.1145/351240.351266.

### 5. Spec-Driven Development: The Emerging Consensus

Patil et al. (2025) demonstrate in an industrial context that spec-first LLM workflows — where requirements are formally structured before generation begins — outperform prompt-and-iterate approaches in correctness and maintainability. The bottleneck, they find, is not generation capability but specification precision: the model can only be as correct as the contract it is given.

DefinitiveSpec is a production-grade implementation of this principle, applying it not just to function-level generation but to the full artifact chain: `goal → requirement → model → design → api → code → test`.

**Key reference:** Patil, M.S., Ung, G., Nyberg, M. (2025). *Towards Specification-Driven LLM-Based Generation of Embedded Automotive Software.* AISoLA 2024. Springer LNCS, vol. 15217. DOI: 10.1007/978-3-031-75434-0_9.

---

## Core Design Principles (The Nine Axioms)

DefinitiveSpec enforces nine axioms at every artifact emission. Violations block output — they are not warnings.

| # | Axiom | What It Prevents |
|---|-------|-----------------|
| 1 | `NO_IMPLEMENTATION_WITHOUT_REPRESENTATION` | Code referencing a library with no `directive` artifact — version drift, undocumented constraints |
| 2 | `CONTEXT_BEFORE_CONTENT` | Design decisions made without researching the target library or API |
| 3 | `EPISTEMIC_HUMILITY` | Assuming library knowledge from training data is current |
| 4 | `SPEC_FIRST` | Raw code generation that bypasses the contract layer |
| 5 | `LOCALITY_OF_BEHAVIOR` | Feature logic scattered across multiple artifacts |
| 6 | `ECONOMIC_RATIONALITY` | Infinite refinement loops with no quality improvement |
| 7 | `DOMAIN_DRIVEN_BOUNDARIES` | Direct mutation of entities that bypass aggregate invariants |
| 8 | `CONTEXT_ECONOMY` | Token waste on irrelevant artifacts in long sessions |
| 9 | `SIMPLICITY_BIAS` | Over-engineering: a 3-component design beats a 7-component one unless justified |

Every axiom is **checked by `<self_critique>` before artifact emission**. No artifact is valid if any check returns `FAIL`.

---

## The Epistemic Marker System

One of DefinitiveSpec's most distinctive features is its **epistemic marker vocabulary** — a set of inline annotations that make the confidence level of every claim explicit and auditable:

| Marker | Meaning | Action Required |
|--------|---------|-----------------|
| `[VERIFIED]` | Backed by a named artifact with a direct quote | None |
| `[INFERRED]` | Logically derived from verified facts | None |
| `[ASSUMPTION]` | Working assumption under incomplete context | Record in `metadata.missing_context`; re-evaluate on artifact load |
| `[UNCERTAIN]` | Missing or ambiguous | Triggers Ghost Artifact or clarification |
| `[BLOCKER]` | Missing critical business rule — cannot be assumed | Hard stop until resolved |
| `[REFUSE]` | Request violates a core rule with no valid workaround | Redesign required |
| `[WARN:tag]` | Detected anomaly (e.g. `NPlusOne`, `BareCATCH`, `Drift`) | Emitted in output; may block emission |

This system eliminates the most dangerous failure mode of LLM-assisted development: silent confidence. A claim that looks authoritative but is actually an unverified hallucination now has a name — `[ASSUMPTION]` — and a mandatory trail.

---

## The Operational Lifecycle

DefinitiveSpec structures work into discrete, gate-controlled phases:

```
Phase 1: Brownfield Sync   →  Map existing code to spec (AST analysis, drift detection)
Phase 2: Bootstrap         →  Load domain index, validate artifact graph
Phase 3: Research          →  BLOCKING: version-lock external libraries via search
Phase 4: Command Focus     →  Identify artifacts, run HealthAnalyzer and StrategicAdvisor
Phase 5: Pre-Generation    →  Guard checks: InvariantViolation, AggregateViolation, SpecConflict
Phase 6: Execution         →  TestDrivenProtocol → ConvergentRefinement → artifact emission
Phase 7: Verification      →  self_critique gate: no FAIL checks may exist on output
```

**Phase 3 is non-negotiable.** Any request involving an external library triggers a search loop capped at 3 searches per library (version lock → working example → known friction). Results are stored as `directive` or `compendium` artifacts. No design that references a library without a corresponding artifact will pass the `NO_IMPLEMENTATION_WITHOUT_REPRESENTATION` check.

---

## Artifact Types

DSpec files are structured YAML with well-defined schemas for each layer of the system:

| Layer | Artifact Types | Purpose |
|-------|---------------|---------|
| **Strategy** | `goal`, `kpi` | Business objectives and measurable outcomes |
| **Requirements** | `requirement`, `glossary` | Functional needs with acceptance criteria |
| **Research** | `research.compendium`, `directive.library` | Version-locked, example-backed library knowledge |
| **Architecture** | `design` | Service responsibilities and dependency graph |
| **Data** | `model` (AGGREGATE_ROOT / ENTITY / VALUE_OBJECT / DTO) | Domain state with type constraints |
| **Contract** | `api`, `code`, `test` | Behavioral contracts with pre/post/invariants |
| **UI** | `user_flow`, `component` | Navigation and composition with slot type-checking |
| **Policy** | `policy.error_catalog`, `policy.nfr`, `policy.security` | Cross-cutting rules and NFR enforcement |
| **Directive** | `directive.library`, `directive.pattern`, `directive.nfr_pattern` | Code-generation instructions |
| **Planning** | `plan` | Task dependency graphs with STALLED detection |
| **Index** | `_domain_index`, `_root_index` | Cross-reference integrity and navigation |

---

## The `logic_blueprint` DSL

The `logic_blueprint` field in `code` artifacts uses a purpose-built pseudo-code vocabulary with **canonical runtime semantics**. Every keyword has a precisely defined observable behavior and failure mode:

```yaml
logic_blueprint: |
  LET order = CALL OrderRepository.findById WITH orderId
  IF NOT (requester.role == ADMIN OR order.ownerId == requesterId) THEN
    RETURN_ERROR error.Unauthorized
  END
  VERIFY order.status != 'DELETED'
  TRANSACTIONAL_BLOCK {
    VERIFY order.version == expectedVersion   # optimistic lock; failure → ConcurrentModification
    SET order.status     = 'DELETED'
    SET order.deleted_at = NOW()
    SET order.version    = order.version + 1
    PERSIST order
    LOG info "Order soft-deleted" { orderId, requesterId }
  }
```

Key semantic guarantees:

- **`TRANSACTIONAL_BLOCK`**: SERIALIZABLE isolation by default. Any exception rolls back all enclosed `PERSIST` calls.
- **`VERIFY`**: Hard assertion with no ELSE path. Failure raises the nearest matching `throws_errors` entry. Unmapped assertions emit `[WARN:UnmappedVerifyError]` at spec time.
- **`PERSIST entity`**: Only valid on `AGGREGATE_ROOT` stereotypes. Direct `PERSIST entity.child` is **forbidden** and emits `[REFUSE: AggregateViolation]`.
- **`FOR_EACH ... { CALL ... }`**: Always triggers `[WARN:NPlusOne]` and requires explicit batching or `[ACCEPTED:NPlusOne reason="..."]`.
- **`LET`**: Immutable binding. Reassignment emits `[WARN:MutableLet]`.

The vocabulary is language-agnostic. Code generators map keywords to platform primitives (e.g. `TRANSACTIONAL_BLOCK` → `BEGIN TRANSACTION` in SQL, `async/await` with rollback in TypeScript).

---

## Convergent Refinement

When generating or improving a spec, DefinitiveSpec applies a **bounded quality optimization loop**:

```
QualityScoreVector dimensions: correctness | clarity | cohesion | nfr_adherence | cost_efficiency
Target: score.primary ≥ 0.85 within 3 cycles, minimum delta 0.02 per cycle

LOOP:
  hypothesis = analyze(score.weakest_metric)
  patch = generate(hypothesis)
  IF patch.touches_security_surface THEN PAUSE for operator review
  apply(patch); score = assess(artifact); cycle++

IF cycle ≥ 3 AND score < target → [STALLED] — human intervention required
```

This replaces the common failure mode of infinite AI refinement loops that produce diminishing returns without converging. When `[STALLED]` is emitted, the system stops and requests explicit human guidance — it does not silently continue generating lower-quality variants.

---

## Pre-Mortem Simulation

Every TIER-2 and TIER-3 artifact passes through a mandatory pre-mortem before emission:

```xml
<pre_mortem status="FINDINGS">
  <check name="RaceConditionCheck" status="WARN">
    Two actors could soft-delete the same order concurrently.
    Mitigation: TRANSACTIONAL_BLOCK checks order.version against expectedVersion.
    Mismatch → RETURN_ERROR error.ConcurrentModification. Applied.
  </check>
  <check name="StateExplosionCheck" status="PASS">Single-record operation.</check>
  <check name="SecurityThreatCheck" status="WARN" owasp_category="A01 Broken Access Control">
    Without role check, any authenticated user could delete any order.
    Mitigation: precondition enforcing ADMIN or ORDER_OWNER. Applied.
  </check>
  <check name="PolicyContradictionCheck" status="PASS">
    Status transition satisfies policy.nfr.DataRetention (no SQL DELETE).
  </check>
</pre_mortem>
```

A `WARN` with a documented mitigation allows emission. A `FAIL` blocks emission unconditionally until the spec is redesigned.

---

## Formal Verification Integration (Optional)

For handlers where `pre_mortem.RaceConditionCheck = WARN` or where multiple aggregates participate in a transaction, DefinitiveSpec proposes a companion **Quint** model-checking spec. Quint (TLA+-compatible semantics with cleaner syntax) allows exhaustive verification of state machine transitions under all possible actor interleavings.

```quint
module DeleteOrder {
  var status: Status
  var version: int

  action deleteOrder(expectedVersion: int): bool = all {
    status != Deleted,
    version == expectedVersion,
    status' = Deleted,
    version' = version + 1
  }

  invariant deletedIsTerminal: status == Deleted implies status' == Deleted
  invariant versionMonotonic: version >= 0
}
```

When `quint verify` passes, the `pre_mortem.RaceConditionCheck` status upgrades from `[WARN]` to `[VERIFIED: quint_spec_path]`. The relationship between DSpec and Quint is unambiguous: DSpec is the source of truth; Quint is the proof artifact. When they diverge, the Quint spec is amended to match the approved DSpec.

---

## Who This Is For

DSpec is not optimized for initial output speed. If the goal is a working prototype in a single session, faster paths exist.

DSpec is for engineers operating under conditions where the cost of a wrong assumption, a missed invariant, or an unverified library dependency is high enough to justify rigorous specification. Concretely:

**Teams that have been burned by AI-generated technical debt** — code that compiles, passes tests, and ships, but contradicts domain invariants that were never written down. DSpec makes invariants explicit and enforces them at the specification layer before code generation begins.

**Projects in regulated domains** — finance, healthcare, government, any context requiring an audit trail from business requirement through architectural decision through implementation contract. Every DSpec artifact carries full provenance.

**Brownfield codebases with established domain models** — where the risk is not "will the AI write correct code" but "will the AI's changes respect the boundaries that took years to establish." Phase 1 Brownfield Sync maps those boundaries before any generation begins.

**Multi-session, multi-contributor projects** — where specification drift is the primary enemy of maintainability. DSpec's DriftDetector treats this as a structural concern, not a review process concern.

**Architects and senior engineers** who want the AI to function as a rigorous specification partner — one whose reasoning is auditable, whose assumptions are tracked, and whose gaps surface as blockers rather than silent guesses.

---

## Getting Started

### Prerequisites

- An LLM session configured with the DefinitiveSpec v9.3 system prompt
- A `specs/` directory in your project root
- Recommended inference temperature: **0.1–0.3** (higher values introduce schema drift)

### Cold Start

If your `specs/strategy/` directory does not exist:

```
[INFO] System is empty. Initiating Cold Start Protocol.
```

DefinitiveSpec will scaffold a `strategy.dspec.yaml` and stop. This is intentional — strategic goals must exist before any feature work begins.

### Your First Artifact

Ask DefinitiveSpec to generate from a natural language requirement:

```
Generate from Requirement: Users must be able to soft-delete their orders.
Order records must be retained for audit purposes.
```

DefinitiveSpec will run its phase sequence and emit a reviewed artifact chain. Here is an excerpt of the `code` artifact you will receive — before a single line of implementation is written:

```yaml
code:
  DeleteOrderHandler:
    language: TypeScript
    signature: "deleteOrder(orderId: UUID, requesterId: UUID, expectedVersion: integer): Promise<void>"
    contracts:
      pre:
        - "orderId is a valid non-null UUID"
        - "requesterId has role ADMIN or is order.ownerId"
        - "order.status is not already 'DELETED'"
        - "expectedVersion matches order.version at time of read"
      post:
        - "order.status === 'DELETED'"
        - "order.deleted_at is set to current timestamp"
        - "order.version is incremented by 1"
      invariants:
        - "No SQL DELETE issued against Order table"
    throws_errors:
      - error.NotFound
      - error.Unauthorized
      - error.ConcurrentModification
```

This contract is the artifact your code generator receives. The implementation is derived from it — not improvised from the original English requirement.

### Reading an Existing Codebase

```
[Brownfield Sync] Please analyze src/handlers/order.ts
```

DefinitiveSpec will map the AST to `code.Signature` and `model.Fields` entries, detect zombie code, and run DriftDetector to compare existing behavior against any previously approved specs.

---

## Comparison with Related Approaches

The SDD ecosystem has several serious entries. This comparison reflects documented architectures and known limitations as of late 2025.

| Approach | What It Does Well | What DefinitiveSpec Adds |
|----------|-------------------|--------------------------|
| **Raw LLM prompting** | Fast first drafts | Formal contracts, pre-mortem, axiom enforcement, **drift detection** (DriftDetector flags when live code diverges from approved spec — the gap that silently accumulates in prompt-driven workflows) |
| **Kiro (AWS)** | Hook-based steering, IDE-native workflow | Typed YAML contracts vs. markdown prose; `[REFUSE]` on AggregateViolation; Phase 3 blocking research loop; native brownfield AST mapping |
| **Tessl** | Spec Registry (10k+ library specs); tests as guardrails | 9-axiom kernel enforced at generation time; assumption tracking with `[WARN:AssumptionViolated]`; full why-stack auditability |
| **TDD** | Test-first discipline | Pre-implementation contract verification, property-based test generation |
| **DDD** | Domain modeling vocabulary | Enforced aggregate boundaries, automated violation detection |
| **OpenAPI / AsyncAPI** | API surface specification | Full artifact chain from goal to test; logic semantics, not just schemas |
| **Formal methods (TLA+, Alloy)** | Mathematical correctness | Human-readable YAML bridge; progressive formality without full proof burden |
| **AI coding agents (Cursor, Copilot)** | Velocity | Correctness: spec-gated generation, invariant verification, no hallucinated libraries |

**A note on fairness:** Tessl's Spec Registry addresses the hallucinated-API problem that DSpec solves via Phase 3 research loops — complementary approaches at different abstraction levels. Kiro's hooks and steering files provide project-level consistency DSpec doesn't attempt to replicate, since DSpec is not an IDE. Teams using any of the above could layer DSpec's constraint kernel as their specification protocol.

---

## Recommended Reading

The following works form the intellectual foundation of DefinitiveSpec:

1. **Hoare, C.A.R.** (1969). *An Axiomatic Basis for Computer Programming.* CACM 12(10), 576–580. — The original formulation of preconditions, postconditions, and program correctness.

2. **Evans, E.** (2003). *Domain-Driven Design: Tackling Complexity in the Heart of Software.* Addison-Wesley. ISBN 0-321-12521-5. — Aggregate roots, bounded contexts, ubiquitous language.

3. **Claessen, K. & Hughes, J.** (2000). *QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs.* ICFP 2000. ACM SIGPLAN, 268–279. DOI: 10.1145/351240.351266. — Property-based testing and counterexample shrinking.

4. **Dijkstra, E.W.** (1976). *A Discipline of Programming.* Prentice-Hall. — Weakest precondition semantics and program derivation.

5. **Lamport, L.** (1994). *The Temporal Logic of Actions.* ACM TOPLAS, 16(3), 872–923. — Foundation for TLA+ and its descendant Quint, used in DefinitiveSpec's formal verification layer.

6. **Ruan, S. et al.** (2024). *Specifications: The Missing Link to Making the Development of LLM Systems an Engineering Discipline.* arXiv:2412.05299. — The central theoretical argument: nearly every class of LLM failure traces back to underspecification. Provides the conceptual foundation for DefinitiveSpec's `contracts` enforcement and `[BLOCKER]`/`[REFUSE]` gate system.

7. **Patil, M.S., Ung, G., Nyberg, M.** (2025). *Towards Specification-Driven LLM-Based Generation of Embedded Automotive Software.* AISoLA 2024. Springer LNCS vol. 15217. DOI: 10.1007/978-3-031-75434-0_9. — Industrial case study demonstrating that spec-first LLM workflows outperform prompt-and-iterate in correctness; validates the phase-gating structure (research → design → contract → generate).

8. **Kambhampati, S. et al.** (2024). *LLMs Can't Plan, But Can Help Planning in LLM-Modulo Frameworks.* arXiv:2402.01817. — Why LLMs require external constraint enforcement rather than self-policing.

---

## License

DefinitiveSpec is a specification methodology. The system prompt and schema definitions are available for use in your own AI-assisted development workflows. Attribution appreciated.

---

## Contributing

DefinitiveSpec improves through adversarial use. If you find a request that produces a specification the nine axioms should have blocked, or a guard that fires incorrectly, please open an issue with:

- The input request
- The artifact produced
- The axiom or guard you believe was violated

The goal is a methodology where every DSpec artifact that passes `<self_critique>` is genuinely trustworthy.

---

*Everything downstream of a specification is derivation. The specification is where rigor must live.*

---

*DefinitiveSpec v9.3 — Constraint Satisfaction Engine for Software Specification*
*`[INFO] Evidence-Based Specification enforced.`*