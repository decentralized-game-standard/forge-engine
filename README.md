# RUNS: Record Update Network System

🏠 **[DGS Overview](https://github.com/decentralized-game-standard)**  
· 📦 **[AEMS](https://github.com/decentralized-game-standard/aems-standard)**  
· ⚡ **[WOCS](https://github.com/decentralized-game-standard/wocs-standard)**  
· 🔤 **[Ludic](https://github.com/decentralized-game-standard/ludic-notation-standard)**  
· ❓ **[FAQ](https://github.com/decentralized-game-standard/.github/blob/main/profile/FAQ.md)**

> **Status**: Draft / RFC  
> **Version**: 0.1.0

## The Neutral Substrate for Enduring Games

**RUNS** (Record Update Network System) is the execution model that enables games to outlive engines, companies, and hardware cycles.

Inspired by Unix pipes and data-oriented design, RUNS treats game engines as composable data-flow graphs: uniform **Records** hold state, stateless **Processors** transform data, and explicit **Networks** wire everything together. Components evolve independently—mods become native extensions, obsolescence in one layer never cascades, and diverse runtimes (minimal web builds to high-performance native) share the same ecosystem like Linux modules.

RUNS is not a full engine or rule language. It is the minimal "kernel" layer: a neutral substrate for transforming data, with game-specific logic implemented in swappable Processors and coordinated across the broader Decentralized Game Standard (DGS).

For extreme longevity and permissionless distribution (e.g., via Nostr relays), distributable artifacts—Networks, Records, and Ecosystem packages—are plain-text by convention, ensuring seamless replication without binaries or central hosting.

## Why RUNS?

Current engines are monolithic and fragile:
- Inheritance of one company's physics, rendering, and fate.
- Mods limited to restricted scripting.
- States tied to dying executables.

RUNS unbundles everything:

- **Universal Modding** — Mods are new Processors or Network extensions with full privileges.
- **Extreme Portability** — Logic runs on any compliant runtime (web, desktop, embedded).
- **Cultural Permanence** — Serializable Records and plain-text Networks allow states to migrate across decades—resume a game long after the original runtime is gone.
- **True Interoperability** — Shared data shapes let Processors from different developers compose freely.
- **Permissionless Ecosystem** — Plain-text packages distribute via Nostr, enabling decentralized discovery and coordination.

## The Mental Model

1. **Records** — The atoms: generic containers (entities) with IDs and Fields of state.
2. **Fields** — The data: typed values on Records (e.g., position, velocity).
3. **Processors** — The logic: pure, stateless transformations reading/writing Fields. Granular like syscalls or hierarchically bundled.
4. **Networks** — The graph: explicit wiring of Processors into data-flow chains. Networks can bundle into higher-scale "meta-Processors" for multi-level composition.
5. **Runtimes** — The executor: loads Networks, manages scheduling, handles input/output.

Example simple loop:

```
[Input Events] → (Input Processor) → [Intent Fields] → (Movement Processor) → [Transform Fields] → (Render Processor) → Screen
```

Swap a naive Movement Processor for a full physics bundle? The chain remains intact as long as shared Fields match.

## What RUNS Enables

- **Multi-Scale Composition** — Primitives wire into mid-level bundles (e.g., velocity integration), which bundle into systems (e.g., character controller), all remaining uniform Processors.
- **Diverse Runtimes** — Minimal interpreters prioritize portability; pro implementations fuse graphs for bleeding-edge performance—all from the same Networks.
- **DGS Integration** — WOCS coordinates Processor bounties; AEMS imports as Records; Nostr distributes plain-text packages permissionlessly.

## What RUNS Deliberately Excludes

RUNS stays restrained and neutral:

- No default renderer, physics, or input model.
- No scripting language—Networks are the composition script.
- No object hierarchy—pure data composition.
- No network transport—focus on local-first state diffs.

## Architecture: Purity in the Protocol, Flexibility in the Library

RUNS separates concerns into three layers to balance eternal neutrality with practical growth.

### Protocol – Non-Negotiables (The Eternal Kernel)

The unbreakable rules ensuring interoperability, longevity, and decentralization. Violate these and it's not RUNS.

- **Record/Field Structure**: Generic, serializable containers (plain-text compatible for state export/import).
- **Processor Interface**: Stateless, pure functions with explicit reads/writes; deterministic; no hidden side effects.
- **Network Format**: Plain-text graph wiring (e.g., JSON/YAML) with hierarchical bundling (sub-Networks as Processors).
- **Execution Model**: Scheduler rules, query language, borrowing safety.
- **Distribution Rules**: Distributable artifacts (Networks, packages) MUST support plain-text formats for permissionless replication (Nostr-ready); no baked binaries in the open chain.
- **Extensibility**: Custom Fields/Processors freely allowed; `runs:` prefix reserved.

This kernel is minimal—like TCP/IP—enabling data to survive centuries and distribute without gatekeepers.

### Standard Library – Recommended Primitives (The Shared Palette)

Curated in a separate repo: [runs-standard-library](https://github.com/decentralized-game-standard/runs-standard-library)

Optional but strongly encouraged schemas and examples for instant interoperability:

- Core Fields like `runs:transform`, `runs:time`, `runs:input`.
- Primitive Processor suggestions (e.g., basic integration ops).
- Best practices for targeting shared shapes.

Use for ecosystem sharing; ignore or extend for custom freedom. Widely adopted primitives emerge organically as the "pigments" for mixing.

### Ecosystem – Where Flexibility Flourishes

Community packages bundling Processors (primitive to full systems):

- Target Protocol for wiring.
- Prefer Standard Library shapes for composability.
- Compete on realizations: naive portable vs. optimized pro.

Packages distribute as plain-text sources/graphs; runtimes compile/fuse locally.

## Why This Distinction Matters

- **Purity without flexibility → dead spec**: The Protocol guards permanence and decentralization.
- **Flexibility without purity → another monolith**: The Library/Ecosystem enable adoption and performance.

Both are vital: one for survival across time, one for thriving today.

## Integration with DGS Standards

| Standard | Role                  | RUNS Relationship                          |
|----------|-----------------------|--------------------------------------------|
| AEMS    | Persistent artifacts  | Imported as Records                        |
| Ludic   | Rule descriptions     | Implemented via Processors                 |
| WOCS    | Coordination/bounties | Funds Processor development and hosting    |

## Comparison

| Feature              | Unity/Unreal          | Bevy/Flecs            | RUNS                          |
|----------------------|-----------------------|-----------------------|-------------------------------|
| Architecture         | Monolithic            | ECS (code-first)      | Data-first neutral substrate  |
| Interop              | Closed                | Language-locked       | Universal plain-text shapes   |
| Modding              | Restricted API        | Compile-time plugins  | Runtime Network injection     |
| Permanence           | Engine-dependent      | Binary-dependent      | Data/Network portability      |
| Distribution         | Stores/platforms      | Git/crates            | Permissionless (Nostr)        |

## Summary

RUNS is the durable socket for decentralized games: explicit, swappable, plain-text native. Combined with DGS layers, it enables experiences that evolve indefinitely without capture.

Implement a Processor. Wire a Network. Build something that endures.

**MIT License** — Open for implementation, extension, critique.