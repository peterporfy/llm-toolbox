# Simple Made Easy — Distilled Principles

Source: Rich Hickey, Strange Loop 2011

## Core Distinction: Simple vs Easy

- **Simple** = one fold, one braid. Not interleaved/entangled with other things. Objective property.
- **Easy** = near at hand, familiar, within comfort zone. Relative to the person.
- **Complex** = braided/folded together. Multiple concepts entangled.
- **Complect** = to interleave, entwine, braid things together. The act that creates complexity.

These are different axes. Something can be easy but complex (familiar ORM patterns). Something can be simple but unfamiliar (a new paradigm that keeps concerns separate).

## Why Simple Matters

- We can only hold a few things in our heads at once (~3-9 "balls in the air").
- Every entanglement forces you to pull additional concepts into working memory.
- Complexity is combinatorial — each interleaving multiplies the cognitive load.
- You cannot reason about, debug, or safely change what you cannot hold in your head.
- Tests and type checkers are guardrails, not guides. Every shipped bug passed all the tests.
- Easy-but-complex gives speed at the start, then progressively kills velocity. Simple gives a slower start but sustained speed.

## Construct vs Artifact

- We program with **constructs** (languages, libraries, patterns we type).
- Users experience **artifacts** (the running system over time).
- Judge constructs by the artifacts they produce, not by how pleasant they are to type.
- Programmer convenience ("I only typed 16 characters!") is not the goal. Long-term maintainability is.

## The Complexity Toolkit (things that complect)

| Complex construct | What it complects | Simpler alternative |
|---|---|---|
| State, variables | Value + time | Values, persistent collections |
| Objects | State + identity + value | Data (maps, sets, sequences) |
| Methods | Function + state + namespace | Functions + namespaces |
| Inheritance | Types with types | Polymorphism à la carte (protocols, type classes) |
| Switch/pattern matching | Who does it + what happens (closed) | Open polymorphism |
| Syntax | Meaning + order | Data |
| ORM | Everything with everything | Declarative data manipulation |
| Imperative loops | What + how + order | Set functions, declarative transforms |
| Actors | What to do + who does it | Queues |
| Conditionals scattered in code | Business rules + program structure | Rule systems, declarative policies |

## Design Principles

### Abstraction = drawing away from the physical
- Not "hiding stuff" — that's encapsulation, a different (lesser) thing.
- Good abstraction: separate the what from the how.
- Use who/what/when/where/why/how decomposition to find entanglements.

### Compose, don't complect
- Composing = placing things together. Independent pieces combined.
- Complecting = braiding things together. Inseparable tangle.
- Modularity alone doesn't guarantee simplicity. Two modules can be deeply complected through hidden assumptions.

### Information is simple — don't ruin it
- Data is maps, sets, and sequences. That's essentially it.
- Wrapping data in classes/objects ties logic to representation.
- Generic data manipulation beats bespoke wrappers.
- Represent data as data. Use maps directly.

### Decouple when/where from what
- Direct calls complect caller with callee's location and timing.
- Queues decouple when and where from what.

### Smaller interfaces, more subcomponents
- Prefer many small, focused abstractions over few large ones.
- Each interface should do one thing.
- Build components via direct injection of subcomponents, not hardwired dependencies.

## The Simplicity Test

When reviewing code, ask:
1. **Is this entangled?** Can I understand/change this piece without pulling in others?
2. **Is this easy or simple?** Am I calling it "simple" because it's familiar, or because it's genuinely unentangled?
3. **What does the artifact look like?** Forget how nice it was to type — what's the running system's complexity?
4. **How many balls?** How many concepts must I hold in my head simultaneously to reason about this?
5. **Is this incidental complexity?** Did the user ask for this, or did our tools/choices introduce it? (Incidental is Latin for "your fault.")
6. **Could this be data?** If you're wrapping information in a class, stop. Use a map.
7. **Could this be a queue?** If A directly calls B, you've complected when/where/who. A queue might untangle it.

## Key Quotes (paraphrased)

- Simplicity is a prerequisite for reliability.
- Simplicity is a choice. It requires constant vigilance.
- Guardrails (tests, types) are safety nets, not substitutes for simplicity.
- We have a culture of complexity — it's self-reinforcing until we consciously break out.
- Simplifying often means ending up with more pieces — but they hang straight instead of being knotted together.
- Simplicity is not about counting. Prefer many straight threads over a few tangled ones.
