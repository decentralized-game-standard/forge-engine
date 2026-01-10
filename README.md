# GERS: Game Engine Record Standard

**A Composable Framework for Game Engine Architecture — Version 0.2 (Conceptual) — January 2026**

GERS is an open standard for building game engines from modular, composable parts. It applies the POSIX philosophy—small tools, universal interfaces, piped composition—to game engine architecture.

*Status: Conceptual. This document defines principles and schemas. No reference implementation exists yet.*

---

## Philosophy: Everything Is a Record

POSIX unified operating systems with one insight: **everything is a file**. Disk, network, device, pipe—all accessed through file descriptors with `read()`, `write()`, `open()`, `close()`.

GERS applies the same insight to game engines: **everything is a Record**.

| POSIX | GERS |
|-------|------|
| File descriptor | Record |
| Byte stream | Field |
| Program | Processor |
| Pipe | Place |
| Pipeline | Network |

A mesh, a sound buffer, an AEMS entity, GPU state—all are Records with Fields. Processors read and write them through a uniform interface. You compose processors into networks like shell pipelines.

---

## Why GERS?

Traditional engines are monoliths. Swap the renderer? Rewrite the engine. Change physics libraries? Good luck.

GERS enables:
- **Modularity**: Swap processors without touching others
- **Composition**: Wire processors like Unix pipes
- **Interoperability**: Standard interfaces mean shared tooling
- **Temporal resilience**: When OpenGL dies, swap the renderer—game survives

---

## Core Concepts

### Record

A handle to any engine resource. Like a file descriptor.

```
record:mesh_001       // A 3D mesh
record:audio_buffer   // Audio samples
record:player_entity  // An AEMS entity
record:gpu_context    // Graphics state
```

Records are identified, not typed. Any processor can attempt to read any record—the Fields determine what's there.

### Field

Data attached to a Record. Like bytes in a file.

```
record:mesh_001
  field:vertices    -> [v0, v1, v2, ...]
  field:indices     -> [0, 1, 2, ...]
  field:transform   -> mat4x4

record:player_entity
  field:position    -> vec3
  field:health      -> int
  field:inventory   -> [item_id, ...]
```

Fields are read and written by Processors. Access semantics (read-only, exclusive, versioned) are declared, not enforced by type.

### Processor

A unit of work that reads Fields, transforms data, writes Fields. Like a Unix program.

```
processor:physics_step
  reads:  [position, velocity, collider]
  writes: [position, velocity]
  emits:  [collision_event]

processor:sprite_render
  reads:  [position, sprite, camera]
  writes: [framebuffer]
```

Processors are:
- **Small**: One job, done well
- **Stateless**: All state lives in Records
- **Declared**: Inputs/outputs specified, not hidden

### Place

A queue where Records wait between Processors. Like a pipe.

```
place:physics_input    // Records waiting for physics
place:render_queue     // Records ready to render
place:audio_pending    // Sounds to play
```

Places decouple Processors. A Processor doesn't know who produced its input or who consumes its output.

### Network

A composition of Processors connected via Places. Like a shell pipeline.

```
network:simple_game
  input_processor -> input_place
  input_place -> physics_processor -> physics_place
  physics_place -> render_processor -> frame_output
```

Networks are declarative. The runtime schedules Processors, respects dependencies, manages timing.

---

## The Orchestra Model

Think of a GERS runtime like an orchestra:

| Orchestra | GERS |
|-----------|------|
| Conductor | Runtime scheduler |
| Score | Network definition |
| Instruments | Processors |
| Sections | Subsystems (rendering, physics) |
| Musicians | Worker threads |
| Rehearsal marks | Sync points |

The conductor (runtime) keeps time. The score (network) specifies who plays when. Each instrument (processor) plays its part without knowing the whole symphony.

This is the opposite of a monolithic engine where one giant `Update()` function does everything sequentially.

---

## Standard Records

Like stdin (0), stdout (1), stderr (2) in POSIX, GERS defines standard Records:

| Record | Purpose |
|--------|---------|
| `record:input` | Player input this frame |
| `record:frame` | Output framebuffer |
| `record:time` | Frame timing (dt, total, fixed) |
| `record:log` | Debug/diagnostic output |

Processors can rely on these existing. Games extend with their own Records.

---

## Concurrency Model

GERS uses Petri net thinking—not as implementation, but as mental model:

- **Places** are queues with tokens (Records)
- **Transitions** are Processors that fire when inputs are ready
- **Concurrent firing**: Processors without conflicts run in parallel
- **Formal reasoning**: Deadlock and starvation are analyzable

This shifts thinking from "sequential update loop" to "concurrent data flow." Implementations may use task graphs, ECS systems, job queues—the paradigm matters more than mechanism.

---

## AEMS Integration

AEMS entities are Records. The mapping:

| AEMS | GERS |
|------|------|
| Entity | A Record type |
| State | Fields on that Record |
| Manifestation | Which Processors apply |

When a game creates an "Iron Sword":
1. AEMS Entity defines what it *is* (universal)
2. AEMS Manifestation defines what it *does here* (this game)
3. GERS Record holds its runtime Fields
4. GERS Processors read/transform/write those Fields

---

## Example: Minimal Game Loop

```yaml
network: minimal_2d

processors:
  - sdl_input:
      writes: [record:input]
      
  - movement:
      reads: [record:input, record:time]
      queries: [position, velocity]
      writes: [position]
      
  - sprite_render:
      reads: [record:time]
      queries: [position, sprite]
      writes: [record:frame]
      
  - present:
      reads: [record:frame]

schedule:
  phases:
    - input: [sdl_input]
    - simulate: [movement]
    - render: [sprite_render, present]
  timing: variable(60fps_target)
```

Four Processors. Each does one thing. The Network wires them. The runtime schedules them.

---

## Prototype Path

| Tier | Action | You Learn |
|------|--------|-----------|
| 1 | Sketch a Network on paper | How Processors connect via Places |
| 2 | Implement 3 Processors in any language | The Processor interface pattern |
| 3 | Build a simple scheduler | How timing and dependencies work |
| 4 | Swap one Processor for another | Modularity via declared interfaces |
| 5 | Add an AEMS Record type | How entities flow through the system |

### Tier 1: Paper Prototype

Draw boxes (Processors) and circles (Places). Draw arrows for data flow. Label the Records that move through. You've designed a GERS Network.

### Tier 2: Three Processors

```python
def input_processor(records):
    records["input"].fields["keys"] = get_keyboard_state()

def physics_processor(records):
    for entity in query(records, ["position", "velocity"]):
        entity["position"] += entity["velocity"] * records["time"].fields["dt"]

def render_processor(records):
    for entity in query(records, ["position", "sprite"]):
        draw(entity["sprite"], entity["position"])
```

### Tier 3: Minimal Scheduler

```python
while running:
    records["time"].fields["dt"] = clock.tick()
    
    for phase in network.phases:
        for processor in phase:
            processor(records)
    
    present(records["frame"])
```

---

## Comparison

| Approach | GERS Equivalent |
|----------|-----------------|
| Unity MonoBehaviour.Update() | Processor in simulate phase |
| Unreal Tick() | Processor in simulate phase |
| ECS System | Processor with query |
| Godot _process() | Processor in simulate phase |

GERS isn't opposed to these—it's a unifying abstraction. An ECS System *is* a Processor. A Unity component's Update() *is* a Processor. GERS just makes the composition explicit and swappable.

---

## Design Principles

1. **Records are universal** — Everything is a Record
2. **Processors are small** — One job, declared interface
3. **Composition is explicit** — Networks wire Processors via Places
4. **Timing is managed** — Runtime respects frame budgets
5. **State is external** — Processors are stateless; state lives in Records

---

## Related Standards

GERS is part of the [Decentralized Game Standard](../../.github/profile/README.md) ecosystem:

| Standard | How It Relates |
|----------|----------------|
| [AEMS](../aems/README.md) | AEMS entities are GERS Records with Fields |
| [WOSS](../woss/README.md) | Processors can be distributed work; compensated via WOSS |

---

## Open Questions

This is a living standard. Unresolved areas:

- **Field access semantics**: How to declare read-only vs exclusive vs versioned?
- **Network hot-reload**: Can you swap Processors at runtime?
- **Distributed processing**: Can Processors run on different machines?
- **Debugging**: How to trace Record flow through Networks?

---

## Contributing

GERS is FOSS. Shape it:
- **Discuss**: Open an issue with questions or ideas
- **Prototype**: Build a minimal GERS runtime
- **Document**: Improve examples and explanations

---

## License

[MIT License](LICENSE) — Use it, extend it, share it.
