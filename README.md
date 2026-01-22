# RUNS: Record Update Network Processor

🏠 **[Overview](https://github.com/decentralized-game-standard)** · 📦 **[AEMS](https://github.com/decentralized-game-standard/aems-standard)** · ⚡ **[WOCS](https://github.com/decentralized-game-standard/wocs-standard)** · ❓ **[FAQ](https://github.com/decentralized-game-standard/.github/blob/main/profile/FAQ.md)**

> **Status**: Draft / RFC  
> **Version**: 0.1.0

## The "Hardware" of Decentralized Gaming

**RUNS** (Record Update Network Processor) defines the execution model for games that live forever.

RUNS reimagines engine architecture as a neutral, data-flow substrate: every resource is a uniform Record, every operation is a small, stateless Processor that reads and writes Fields, and composition happens through explicit Networks of queued data. The result is an execution environment that evolves component-by-component without systemic collapse. Mods become native extensions. Obsolescence in one layer never dooms the whole. Diverse engines—from minimalist hobbyist builds to high-performance "pro" distributions—can emerge from the same standard, swapping components freely like Linux modules built by hundreds of contributors.

RUNS is not a rule-description language or a specific game framework. It is the standardized "hardware + kernel" layer: the runtime that transforms data efficiently, leaving game-specific behaviors (core loops, win conditions, mechanics) to be implemented in swappable Processors and described separately via complementary standards.

## Why RUNS?

Current game engines are monoliths. If you build on Unity, you inherit Unity's physics, Unity's rendering assumptions, and Unity's corporate destiny. If you mod a game, you hack a compiled binary or script against a restricted API.

RUNS unbundles the engine.

- **Universal Modding** — A "mod" is just a new Processor wiring into the Network. It has the same privileges as the base game code.
- **Portability** — Game logic defined as a Network running on standard Records works on any RUNS-compliant runtime (Web, Desktop, Mobile, Terminal).
- **Permanence** — Records are serializable data. A game state can be saved, moved to a new runtime, and resumed 10 years later, even if the original "engine" executable is dust.
- **Interoperability** — Because data shapes are standardized (via the [RUNS Standard Library](../runs-standard-library/README.md)), a "Physics Processor" from one developer can drive the "Rendering Processor" of another.

RUNS draws from Unix pipes and data-oriented design: small transformations applied to explicit streams of well-layout data.

## The Mental Model

1.  **Records**: The atoms. Generic containers of data (Entities). They have IDs and Fields. They don't have methods.
2.  **Fields**: The state. Data attached to Records. `runs:transform`, `runs:velocity`, `aems:durability`.
3.  **Processors**: The logic. Pure functions causing side effects. `Move(runs:transform, runs:velocity)`. Processors are stateless and interchangeable.
4.  **Networks**: The motherboard. A graph defining how data flows between Processors.
5.  **The Runtime**: The electricity. It loads the Network, feeds it input events, and manages the loop.

## Example: A Simple Loop

```
[Input Event] -> (Input Processor) -> [Control Data] -> (Movement Processor) -> [Transform Data] -> (Render Processor) -> Screen
```

Replace (Movement Processor) with a complex Physics engine? The rest of the chain doesn't care, as long as `runs:transform` comes out the other side.

## What RUNS Enables

- **WOCS Integration** — Processor development can be coordinated via bounties. "I need a better Pathfinding Processor for this RTS." A dev writes it. Work is settled instantly via WOCS.
- **Diverse Engines** — One minimal RUNS build targets embedded devices; another optimizes for massive multiplayer; a third prioritizes high-fidelity rendering—all sharing the same Processor ecosystem.

## What RUNS Deliberately Excludes

RUNS is an execution substrate, not a game engine. Like POSIX defines OS interfaces without dictating implementation, RUNS defines data-flow patterns without prescribing behavior:

1.  **No Default Renderer**: RUNS defines *how* to pass data to a renderer, but not *which* renderer to use. (Processors wrap WGPU, Raylib, ASCII code, etc.)
2.  **No Scripting Language**: RUNS networks *are* the script. Processors can be written in Rust, C, AssemblyScript (Wasm), etc.
3.  **No "Game Object" Hierarchy**: There is no inheritance. There is only composition of Data Fields on Records.
4.  **No Network Transport**: RUNS defines how specific node synchronization happens (via state diffs), not TCP/UDP implementation.

A RUNS-conformant engine could be a minimal hobbyist runtime or a high-performance commercial product. The standard specifies the interface contract; everything else is competitive differentiation.

## Architecture

To achieve universal support for all simulation types—from discrete turn-based logic (Chess, Go) to continuous high-frequency simulations (FPS, Physics)—RUNS separates the standard into three distinct layers: the Protocol (The Machine), the Standard Library (The Vocabulary), and the Ecosystem (The Logic).

This separation removes all genre-specific assumptions (spatial, temporal, or input-based) from the core, ensuring RUNS is a truly neutral substrate for decentralized computation.

### Layer 1: The RUNS Protocol ("The Machine")
**Repository**: `runs-protocol` (Abstract Definition)

The Protocol is the unopinionated kernel. It defines **only** the mechanics of execution and data flow. It knows nothing about "Time", "Space", or "Input".

**Responsibilities**:
- **Data Layout**: Defines the binary structure of `Record` (Entity), `Component` (Data Table), and `Archetype` (Component Set).
- **Execution Model**: Defines the `System` (Function) signature and the `Scheduler` (Graph) rules.
- **Query Language**: Defines how Systems request data (`Query<(&Transform, &Velocity)>`).
- **Safety**: Defines borrowing rules to prevent race conditions (Rust-style borrow checker adapted for ECS).

**Key Constraint**: The Protocol must be implementable in any language (Rust, C++, Zig, TS), but Wasm compatibility is P0.

- **Reservation**: IDs starting with `runs:` are reserved for Protocol and Standard Library usage.

### Layer 2: The RUNS Standard Library ("The Vocabulary")
**Repository**: `runs-standard-library`

The Standard Library provides the **Semantic Agreement** necessary for interoperability. While the Protocol says "Here are bytes," the Standard Library says "These bytes represent a Position."

It is a collection of standardized Components and Resources that engines *should* support to be compatible with the wider ecosystem.

**Content**:
- **Core Schemas**: `runs:time` (DeltaTime, Uptime), `runs:input` (key_code, axis), `runs:lifecycle` (State).
- **Spatial Schemas**: `runs:transform` (Position, Rotation, Scale), `runs:hierarchy` (Parent/Child).
- **Meta Schemas**: `runs:name`, `runs:tag`.

See [RUNS Standard Library](../runs-standard-library/README.md) for full definitions of:
- `runs:root`
- `runs:time`
- `runs:transform`
- `runs:input`

### Layer 3: The RUNS Ecosystem ("The Logic")
**Repository**: Community / Marketplace

This is where the actual game code lives. "Engines" in the traditional sense (Unity, Godot) are actually just **Bundles** of Ecosystem implementations.

**Definition**:
An "Ecosystem Package" is a set of Systems (Logic) that:
1. MUST target the RUNS Protocol for wiring.
2. SHOULD read/write schemas defined in the RUNS Standard Library.
3. MAY define its own custom Component schemas for internal logic.

**The Power of Separation**:
Because `std:spatial` defines `runs:transform` (the data shape), we can have competing implementers of logic:

- **Package A (simple-mover)**: Reads `std:input`, Writes `runs:transform`. (Directly modifies position).
- **Package B (rigid-body-physics)**: Reads `runs:transform`, Calculates Forces, Writes `runs:transform`.

A user can swap Package A for Package B. The "Game Object" (Record) does not change; it still just has a `runs:transform`. The behavior changes, but the data remains standard.

## Integration with Standards

| Standard | Role | RUNS Relationship |
| :--- | :--- | :--- |
| **AEMS** | The "Matter" (Asset Definitions) | RUNS engines ingest AEMS Entities as Records. |
| **Ludic** | The "Blueprint" (Game Rules) | RUNS Processors implement the verbs defined in Ludic structures. |
| **WOCS** | The "Coordination" (Ecosystem Services) | Server hosting, anti-cheat bounties, and Processor development are coordinated via WOCS. |

## Comparison

| Feature | Unity/Unreal | Bevy/Flecs | RUNS |
| :--- | :--- | :--- | :--- |
| **Architecture** | Monolithic (OOP + Components) | ECS (Code-First) | Data-First Protocol |
| **Interop** | None (Closed Eco) | Binary/Language locked | Universal Data Shape |
| **Modding** | Scripting API (Restricted) | Plugin (Compile time) | Network Injection (Runtime) |
| **Networking/multiplayer** | Platform services | Library specific | Protocol is local-first |
| **Monetization** | Asset Store (Closed) | GitHub Sponsors | WOCS (Direct Coordination) |

## Summary

**RUNS** is the socket.
**AEMS** is the bulb.
**WOCS** is the coordination layer for upkeep—server hosting, anti-cheat, provenance audits.

RUNS is a pattern for engines that endure: explicit, measurable, replaceable—serving as the neutral substrate for diverse, evolving game experiences.

Experiment. Implement a single Processor. Watch how cleanly it composes.

**MIT License** — Open for implementation, extension, critique.