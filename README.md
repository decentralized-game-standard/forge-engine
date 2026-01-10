# GERS: Game Engine Record Standard

**A Composable, Data-Oriented Architecture for Resilient Game Engines — Version 0.2 (Conceptual) — January 2026**

Game engines have become the foundational infrastructure of interactive software, yet most are built as deeply intertwined monoliths. A change in one subsystem—renderer, physics, input—often cascades into widespread breakage. Upgrading a graphics API can require rewriting core loops. Adding a new feature risks polluting unrelated code. Performance bottlenecks hide behind layers of inheritance and virtual calls.

This is not inevitable. It stems from design choices that prioritize apparent convenience over explicit control and long-term maintainability.

The Game Engine Record Standard (GERS) reimagines engine architecture through a data-flow model: every resource is a uniform Record, every operation is a small Processor that reads and writes Fields, and composition happens through explicit Networks of queued data. The result is an engine that can evolve component-by-component without systemic collapse, where mods are native extensions, and obsolescence in one layer does not doom the whole.

## Diagnosing the Monolithic Trap

Modern engines like Unity and Unreal demonstrate both extraordinary capability and structural rigidity.

- **Version Lock-In** — Projects often remain pinned to specific engine versions for years because upgrades break serialized data, plugins, or shader pipelines. Unity's 2021–2022 upgrade cycle introduced physics regressions and asset incompatibilities that forced many teams to delay or fork. Similar stories recur with every major Unreal point release: blueprint graphs, animation state machines, or material graphs subtly change behavior.

- **Abstraction Overhead** — Object-oriented designs dominant in these engines encourage deep hierarchies and virtual dispatch. Jonathan Blow has repeatedly highlighted how this fragments data layout, causing cache misses that dominate frame time in large scenes. A typical Unity MonoBehaviour update loop scatters position, velocity, and transform data across unrelated objects, forcing the CPU to chase pointers instead of streaming contiguous arrays.

- **Iteration Friction** — Compile times in C++-based engines routinely exceed minutes for trivial changes. Blow's critiques center on this: layers of headers, templates, and precompiled dependencies turn rapid prototyping into a slog, discouraging experimentation and deep optimization.

- **Hidden Costs and Fragility** — Monolithic update loops entangle subsystems. Input processing might inadvertently touch rendering state. A plugin overriding one behavior can corrupt another. When APIs deprecate (DirectX 11 to 12, OpenGL's decline), the entire engine must adapt at once, stalling projects.

- **Modding as Afterthought** — Mods typically patch binaries or inject scripts, fragile against updates. Skyrim's Script Extender exists precisely because Bethesda's engine offers no clean extension points.

These issues compound over time. Engines accrete features to serve the widest audience, becoming harder to maintain, debug, or escape.

## Core Insight: Data Flow Over Control Flow

GERS draws from Unix's proven model—small tools reading and writing byte streams through pipes—and from data-oriented design principles that prioritize cache-efficient layouts and explicit transformations.

Everything in a GERS engine is a **Record**: an opaque handle to a bundle of typed **Fields**. A mesh, an entity, input state, or a framebuffer—all are Records. There is no inheritance hierarchy; meaning emerges from the Fields present and the Processors that interpret them.

**Processors** are stateless functions that declare inputs and outputs:

- They read specific Fields from input Records.
- They write or emit new Fields/Records.
- They contain no hidden global state.

**Places** are queues decoupling Producers from Consumers, enabling concurrency without locks.

A **Network** declaratively wires Processors via Places, defining the engine's topology.

This shifts the mental model from "objects sending messages" to "data flowing through transformations." It exposes costs explicitly—no virtual calls hiding cache thrashing, no accidental dependencies.

### Why This Enables Performance and Longevity

Data-oriented layouts (arrays of structs over structs of arrays) become natural: a Processor querying all "position" and "velocity" Fields receives contiguous memory. Parallel scheduling across cores is straightforward—independent Processors fire simultaneously.

Swapping subsystems is declarative: replace a DirectX Renderer Processor with a Vulkan one; the inputs (mesh Records, transform Fields) remain identical.

Modding becomes integration: a mod author publishes a new Processor (e.g., advanced AI decision making) that slots into the Network at defined points.

## Technical Foundations

### Record
An identifier referencing mutable or immutable data.

```
record:player_42
  field:position    -> vec3
  field:velocity    -> vec3
  field:health      -> i32
  field:inventory   -> array<record>
```

Records carry no behavior—only data.

### Field
Typed values attached to Records. Access is versioned or exclusive as declared.

### Processor
Declares read/query/write signatures.

```
processor:integrate_motion
  reads:  [time.dt]
  queries: [position, velocity]  // all Records possessing both
  writes: [position]
```

Implementation is plain code operating on bulk data.

### Place
A typed queue of Records.

```
place:simulation_output -> place:render_input
```

### Network
YAML-like declaration (runtime-agnostic):

```yaml
network: core_loop
  processors:
    - input_processor    -> place:input
    - motion_integrator  -> place:physics
    - collision_detector -> place:events
    - renderer           -> record:framebuffer
  schedule:
    - phase: input
    - phase: simulation [parallel]
    - phase: render
```

The runtime resolves dependencies, respects timing, and parallelizes phases.

### Concurrency Model
Inspired by Petri nets: Places hold tokens (Records), Processors fire when inputs available. No manual threading—runtime manages task graphs or job systems.

### Standard Records
Minimal conventions for interoperability:

- `record:time` — dt, elapsed, fixed_step
- `record:input` — devices, events
- `record:frame` — output buffer
- `record:log` — diagnostics

## Integration with Broader Ecosystem

AEMS entities map directly to Records: the universal Entity defines expected Fields, Manifestations guide which Processors apply, State updates Fields, ownership is external.

A modder paid via WOSS might deliver a new Processor enhancing lighting for specific AEMS Manifestations.

The engine remains agnostic to persistence—Records can serialize to Nostr events or local files.

## Example: Simple 2D Pipeline

```yaml
processors:
  - capture_input:
      writes: [record:input]
  - apply_forces:
      reads: [record:input, record:time]
      queries: [position, velocity, controller]
      writes: [velocity]
  - integrate:
      reads: [record:time]
      queries: [position, velocity]
      writes: [position]
  - render_sprites:
      reads: [record:time]
      queries: [position, sprite, camera]
      writes: [record:frame]
```

Four focused components. Replace `render_sprites` with a ray-traced variant—no other code changes.

## Prototype Path

1. **Paper Design** — Sketch Records and Processors for a minimal game.
2. **Single-Thread Prototype** — Implement three Processors in any language, manually sequencing them.
3. **Scheduler** — Add dependency resolution and phasing.
4. **Parallel Execution** — Introduce job system for concurrent phases.
5. **AEMS Import** — Load an external Entity as a Record, apply a Manifestation via Processor selection.

## Why Pursue This Now

Hardware trends favor wide, cache-coherent processing. Languages and tools increasingly support data-oriented patterns. The costs of monolithic lock-in are evident in stalled projects and abandoned engines.

GERS is not a rejection of existing engines but a pattern for building ones that endure: explicit, measurable, replaceable.

Experiment. Question the declarations. Implement a single Processor and see how cleanly it composes.

**MIT License** — Open for implementation, extension, critique.