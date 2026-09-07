# bonsai/intent

Intent ontology and intermediate representation for Bonsai agents.

## Purpose

**Intent is the semantic layer between human goals and executable agents.**

```text
Intent
  ↓
Goal / Context / Constraints
  ↓
Plan
  ↓
Agent
  ↓
Workflow / AW
  ↓
Action
  ↓
Artifact
  ↓
Evidence / Outcome
```

Intent answers: **who wants what, why, against which object, under which constraints, and what outcome counts as success?**

## Core ontology

| Concept | Meaning |
|---|---|
| `Intent` | A directed purpose to realize an outcome |
| `Actor` | Entity holding or executing an intent |
| `Goal` | Desired state or objective |
| `Object` | Target of the intent |
| `Context` | Situation in which the intent exists |
| `Constraint` | Rule, boundary, or requirement |
| `Action` | Executable operation |
| `Plan` | Ordered strategy for realizing an intent |
| `Outcome` | Result produced by execution |
| `Evidence` | Observation supporting an intent or outcome |

## Relations

```text
Actor ──hasIntent──────► Intent
Intent ──intendsGoal───► Goal
Intent ──targets───────► Object
Intent ──occursIn──────► Context
Intent ──constrainedBy─► Constraint
Intent ──plannedBy─────► Plan
Intent ──realizedBy────► Action
Action ──produces──────► Outcome
Intent ──supportedBy───► Evidence
```

## Lifecycle

```text
PROPOSED → UNDERSTOOD → ACCEPTED → PLANNED → EXECUTING → COMPLETED
                                                       ├→ SUCCEEDED
                                                       └→ FAILED
```

Additional terminal state: `CANCELLED`.

## Intent as Agent IR

Bonsai treats Intent as an intermediate representation for agent generation:

```text
Natural language / Repository
            ↓
        Ontology
            ↓
        Intent IR
            ↓
          Agent
            ↓
        AW Workflow
```

This enables `repo2agent`: repository structure and README semantics can be interpreted into intents, which can then be compiled into agents and workflows.

## Example

```yaml
intent:
  id: repo.health.monitor
  status: proposed

  actor:
    type: agent
    role: repository-observer

  goal:
    type: monitor
    object: repository-health

  targets:
    - repository: bonsai/repos

  constraints:
    - recent-repositories-only
    - reproducible
    - evidence-required

  actions:
    - collect
    - measure
    - classify
    - report

  outcomes:
    - health-score
    - organization-graph
    - evidence-report
```

## Repository layout

```text
ontology/
  intent.ttl
  intent.yaml
  intent.jsonld
concepts/
  intent.md
  goal.md
  actor.md
  action.md
  outcome.md
  evidence.md
schemas/
  intent.schema.json
examples/
docs/
```

The canonical model may be serialized as YAML, JSON-LD, or RDF/Turtle without changing the conceptual ontology.

## Bonsai architecture

`bonsai/intent` sits above execution systems:

```text
intent
  ↓
yaml-as-agent
  ↓
agent
  ↓
aw
```

Repository intelligence can feed the same model:

```text
repos → ontology → intent → agent → workflow → evidence → outcome
```
