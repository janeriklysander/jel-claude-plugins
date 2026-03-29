---
name: ddd-design
description: Use when the user wants to model a domain, design bounded contexts, run a DDD session, do event storming, establish ubiquitous language, or design a domain model for a new or existing system.
---

# DDD Design

## Overview

Run a structured Domain-Driven Design discovery session. Explore the codebase for existing domain concepts, then interview the user through strategic then tactical DDD concerns — one question at a time — pushing back on modelling mistakes. Produce a purely conceptual domain design document at `docs/domain/<name>.md`.

## Process

### Phase 1: Codebase Exploration

Before asking anything, explore the codebase for implicit domain concepts already present:

- Entity and model names
- Service and repository names
- Event names and message types
- Folder/module structure that suggests domain boundaries

Note what you find — these become anchors in the interview so the model doesn't conflict with existing code.

### Phase 2: Interview

Work through the checklist below in order. **Ask exactly one question at a time.** Wait for the answer before proceeding. For each question, provide your recommended answer based on what you've heard so far.

**Act as a domain expert collaborator — push back when:**
- An aggregate should be a value object (or vice versa)
- A context boundary will create tight coupling
- Terminology is ambiguous or overloaded
- A "core" subdomain is actually generic (e.g. auth, email, payments via third-party)

State your concern clearly, then let the user override.

#### Strategic Layer

1. **Domain name** — What is this system's domain called? What problem does it solve in one sentence?
2. **Actors** — Who are the key actors (users, systems, external parties)?
3. **Business processes** — What are the main workflows or business processes?
4. **Ubiquitous language** — What terms does the business use? Are any terms overloaded or ambiguous across teams?
5. **Subdomains** — What are the distinct problem areas? Classify each: Core / Supporting / Generic.
6. **Bounded contexts** — What are the named contexts, and what is each responsible for?
7. **Context map** — How do contexts relate? (upstream/downstream, shared kernel, anti-corruption layer, open host service, conformist)

#### Tactical Layer (per bounded context)

8. **Aggregates** — What are the aggregate roots in each context? What consistency boundary does each protect?
9. **Entities vs value objects** — For each concept within an aggregate, does identity matter (entity) or is it defined purely by its attributes (value object)?
10. **Domain events** — What past-tense events does the business care about? Who produces and who consumes each?

#### Wrap-up

11. **Open questions** — What remains unresolved or needs further discovery?

### Phase 3: Generate Document

Once all checklist items are resolved or explicitly deferred, write the document to `docs/domain/<domain-name>.md`. Create the directory if it doesn't exist.

## Output Format

```markdown
# Domain: <Name>

> <One paragraph: the problem this domain solves>

## Ubiquitous Language

| Term | Definition |
|------|------------|
|      |            |

## Subdomains

| Subdomain | Type | Description |
|-----------|------|-------------|
|           | Core / Supporting / Generic | |

## Bounded Contexts

### <Context Name>

**Responsibility:** ...
**Key concepts:** ...

## Context Map

| Relationship | Type | Notes |
|--------------|------|-------|
| A → B | Upstream/Downstream | |

## Aggregates

### <Context Name>

#### <Aggregate Root>

- **Entities:** ...
- **Value Objects:** ...
- **Invariants protected:** ...

## Domain Events

| Event | Producer | Consumers | Trigger |
|-------|----------|-----------|---------|
|       |          |           |         |

## Open Questions

- [ ] ...
```

## Rules

- **One question at a time.** Never bundle questions.
- **Push back actively.** Flag modelling mistakes with a reason. The user can override.
- **No implementation details.** No folder structures, code patterns, or framework choices — the document must be readable by a non-technical domain expert.
- **Skill drives completion.** Work through the full checklist, then write the document. Do not ask "are we done?"
