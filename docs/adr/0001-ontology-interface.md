# ADR-0001: Ontology Interface Boundaries

- Status: Accepted
- Date: 2026-09-07
- Decision: Connect ontologies through explicit semantic interfaces and mappings rather than merging them into one ontology.

## Context

Bonsai contains multiple semantic domains:

- `bonsai/intent` — intent, goal, constraint, plan, action, outcome, evidence
- `bonsai/world-ontology` — world/domain concepts and context
- `bonsai/wiki` — human-curated concepts and knowledge
- `bonsai/repos` — repository state, facts, health, and organization data
- domain repositories — specialized vocabularies and agent concepts

These repositories need to interoperate without creating a single large ontology. A direct merge would make ownership unclear, increase coupling, and make independent evolution difficult.

The important boundary is therefore not the repository boundary itself, but the **semantic interface between ontologies**.

## Decision

Ontologies SHALL connect through an explicit **Ontology Interface**.

The interface has four layers:

```text
Source Ontology
      │
      │ provides concepts / facts
      ▼
Semantic Interface
      │
      ├── vocabulary mapping
      ├── identifier / URI mapping
      ├── schema / shape contract
      └── provenance / evidence
      │
      ▼
Consumer Ontology
```

### 1. Keep ontology ownership separate

Each repository owns its vocabulary and meaning.

`bonsai/intent` owns intent semantics. It does not redefine repository, person, product, domain, or other world concepts merely to consume them.

### 2. Connect by stable identifiers

Concepts crossing a repository boundary SHOULD have stable identifiers, preferably URIs or namespaced IDs.

Example:

```text
https://bonsai.dev/intent#Intent
https://bonsai.dev/world#Repository
https://bonsai.dev/repos#RepositorySnapshot
```

### 3. Use mappings instead of duplication

Cross-ontology equivalence or specialization SHALL be expressed as a mapping.

Typical mapping relations:

```text
exactMatch
closeMatch
equivalentClass
subClassOf
sameAs
relatedTo
```

A mapping belongs at the interface boundary, not inside the unrelated source ontology.

### 4. Separate semantics from transport

The semantic contract is independent of serialization or transport.

```text
Ontology Interface
       │
       ├── RDF / Turtle
       ├── JSON-LD
       ├── YAML
       ├── JSON Schema
       └── API / file / Git commit
```

Changing YAML to JSON-LD, for example, MUST NOT change the intended meaning of the interface.

### 5. Evidence crosses the boundary with provenance

Facts imported from another ontology or repository MUST retain provenance when practical.

```text
Fact
 ├── source
 ├── observedAt
 ├── subject
 ├── predicate
 ├── object
 └── evidence
```

This prevents an inferred fact from being confused with an authoritative domain definition.

## Bonsai interface map

```text
                 ┌──────────────────────┐
                 │    world-ontology    │
                 │ world / domain model │
                 └──────────┬───────────┘
                            │ context / object
                            ▼
┌──────────────┐      ┌──────────────────┐      ┌──────────────┐
│     wiki     │─────▶│  bonsai/intent   │◀─────│    repos     │
│ knowledge    │      │ semantic control │      │ observations │
└──────────────┘      │      plane       │      └──────────────┘
                      └────────┬─────────┘
                               │ intent / plan
                               ▼
                      ┌──────────────────┐
                      │      think       │
                      │ reasoning/plan  │
                      └────────┬─────────┘
                               │ agent definition
                               ▼
                      ┌──────────────────┐
                      │  yaml-as-agent   │
                      └────────┬─────────┘
                               │
                               ▼
                      ┌──────────────────┐
                      │      agent       │
                      └────────┬─────────┘
                               │ executable workflow
                               ▼
                      ┌──────────────────┐
                      │       aw         │
                      └────────┬─────────┘
                               │ execution evidence
                               ▼
                      ┌──────────────────┐
                      │    ds-agent      │
                      │ analysis/eval    │
                      └──────────────────┘
```

## Interface vocabulary

The Bonsai repository graph SHOULD use these relations consistently:

| Relation | Meaning |
|---|---|
| `defines` | owns the authoritative semantic definition |
| `provides` | exposes concepts, data, or contracts |
| `consumes` | depends on another ontology's interface |
| `mapsTo` | explicitly maps one concept to another |
| `specializes` | narrows or extends another concept |
| `observes` | produces observations about an object |
| `analyzes` | derives metrics or inference from observations |
| `plans` | converts intent into an executable strategy |
| `generates` | creates an agent or workflow definition |
| `executes` | performs an action or workflow |
| `verifies` | evaluates an outcome against evidence |

## Canonical flow

The intended cross-ontology lifecycle is:

```text
observe
  ↓
map
  ↓
understand
  ↓
intend
  ↓
plan
  ↓
generate
  ↓
execute
  ↓
collect evidence
  ↓
evaluate
  ↓
re-intend
```

This makes ontology integration a controlled feedback loop rather than a one-time schema conversion.

## Consequences

### Positive

- Repositories retain semantic ownership.
- Ontologies can evolve independently.
- `bonsai/intent` remains a stable semantic control plane.
- `repo2agent` can use mappings as explicit compilation boundaries.
- Provenance makes generated decisions auditable.
- New domain ontologies can join the ecosystem without modifying the core ontology.

### Negative

- Mapping files and interface contracts must be maintained.
- Some concepts will remain intentionally ambiguous and require `closeMatch` or `relatedTo` rather than forced equivalence.
- Tooling is needed eventually to validate interface compatibility.

## Rejected alternatives

### A. One global Bonsai ontology

Rejected. It centralizes ownership and creates excessive coupling between repositories.

### B. Direct schema-to-schema conversion

Rejected. Serialization compatibility does not guarantee semantic compatibility.

### C. Implicit name matching

Rejected. Identical names can have different meanings; semantic identity must be explicit.

### D. API-only integration

Rejected. An API is a transport mechanism, not an ontology interface. The semantic contract must remain independently defined.

## Implementation direction

The next implementation steps are:

1. Add `docs/ecosystem.md` describing repository-level interfaces.
2. Add mapping artifacts under `ontology/mappings/`.
3. Define JSON Schema / SHACL-style validation for interface payloads.
4. Give each connected ontology stable namespace identifiers.
5. Make `repo2agent` consume mappings rather than guessing semantic equivalence.
6. Let `bonsai/aw` execute only intents that satisfy the declared interface contract.

## Principle

> **Ontologies should compose at their boundaries, not collapse at their centers.**

`bonsai/intent` is the semantic interface layer that connects the Bonsai ontology ecosystem while preserving the autonomy of each ontology.