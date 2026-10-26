---
title: Deterministic Simulation
description: The boundaries that keep SeinARTS lockstep state reproducible across peers and platforms.
---

Determinism in SeinARTS is an architectural boundary. The authoritative simulation produces the same state from the same initial state and ordered command stream on every peer.

## Fixed-point state

Simulation math uses 32.32 fixed-point types:

- `FFixedPoint` for scalar values.
- `FFixedVector` for positions and directions.
- `FFixedTransform` and `FFixedQuaternion` for transforms and rotation.
- `FFixedRandom` for deterministic pseudo-random values.

Simulation code uses `FSeinEntityHandle` instead of raw Unreal object or actor pointers. Floating-point math, `FVector`, Unreal random helpers, and renderer-owned objects stay outside authoritative state. A `float` or `double` field in a simulation type stops the match from starting, and a Blueprint compile warns about it first. Mark a field Presentation when it only feeds visuals.

## Simulation and presentation

Data crosses the boundary in one direction:

```text
Input → commands → deterministic simulation → presentation
```

The player controller and AI controllers submit commands. They do not change simulation storage directly. Presentation reads the simulation to animate, display and play effects, but nothing it computes becomes a simulation input.

## One world state

Everything the simulation owns lives in one world state: the entities and their components, tags, ability and effect objects, players and the match, module state such as navigation's pending paths, and the random stream. Caches such as the spatial index are rebuilt from it.

One serializer writes that state. The same bytes serve every job that needs the match as data:

- Save and load, mid-match included.
- Bringing a reconnecting player, or a player that drifted out of sync, back to the host's state.
- Replay checkpoints for seeking.
- The checksum peers compare to detect a desync.

New simulation data joins all of these automatically when it is a component, a property of an ability or effect, or part of a module's registered state.

## Configuration is state

Project settings that change the simulation result are hashed. A player whose build, settings or map differ from the host's cannot join the match, instead of drifting out of sync later.

## Proving determinism

A successful compile cannot prove deterministic behavior. Simulation changes are checked by running the same scenario serially and in parallel and comparing every tick's checksum, by loading a saved state into a fresh world and comparing it with an uninterrupted run, and by networked and replay sessions.
