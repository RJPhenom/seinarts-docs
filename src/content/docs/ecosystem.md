---
title: Plugin Ecosystem
description: The production plugins, extension boundaries, and dependency rules that make up SeinARTS.
---

SeinARTS ships as one core plugin and one opt-in production extension. Dependencies point toward the framework, never back from the framework into an extension.

## Production plugins

| Plugin | Adds | Requires |
| --- | --- | --- |
| **SeinARTSFramework** | Deterministic simulation, entities, abilities, effects, navigation, base movement, fog of war, networking, editor tooling, UI, and gameplay shell | Nothing else in SeinARTS |
| **SeinARTSOnlineServicesExtension** | Provider-neutral account, party, matchmaking, results, saves, replay evidence, telemetry, and provider adapters | Framework |

## Dependency shape

```text
SeinARTSFramework
└── SeinARTSOnlineServicesExtension
```

Game-specific features, such as persistent unit groups, scored standing positions or vehicle movement modes, belong to the game. A game builds them in its own modules on the framework's public seams: broker resolvers and formations, destination plan providers, simulation systems and entity hooks, and movement modes.

## Development-only test plugins

Two disabled test plugins validate the ecosystem without becoming shipping dependencies:

- **SeinARTSTestSuite** exercises the framework and editor surfaces.
- **SeinARTSExtensionTestSuite** consumes the base test suite and every opt-in extension for integration coverage.

No production plugin may depend on either test plugin.

## Shared contracts

Every production plugin follows the same boundaries:

- Simulation state flows to rendering; rendering does not write authoritative state.
- Simulation uses deterministic fixed-point math and stable entity handles.
- Designer-authored Blueprint content composes deterministic C++ primitives.
- Settings that change the simulation are hashed; peers with different values cannot join a match.
- Optional features remain removable without breaking the framework.

