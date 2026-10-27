---
title: Getting Started
description: Understand the SeinARTS architecture and choose the plugins your project needs.
---

SeinARTS is a deterministic lockstep RTS framework for Unreal Engine 5. Its C++ layer provides deterministic primitives and infrastructure; designers assemble game-specific behavior in Blueprint.

:::caution[Prerelease]
SeinARTS is distributed to licensees as prebuilt plugin packages for Unreal Engine 5.8 on Win64. The 0.3 releases are prereleases: APIs, saves and replays may change between releases without migration.
:::

## 1. Start with the framework

Every SeinARTS project uses **SeinARTSFramework**. Its modules cover the simulation core, collision, level data, navigation, fog of war, abilities and effects, orders and formations, the economy, the match, base movement, networking, presentation, the Unreal gameplay classes, UI, debugging and editor tooling.

The framework does not depend on any SeinARTS extension. Removing every optional extension leaves a valid core plugin.

## 2. Select optional capabilities

Enable only the extensions your game needs:

- **Online Services** for provider-neutral account, party, matchmaking, results, saves, replay evidence, and telemetry contracts.

Features specific to your game, such as persistent unit groups, scored standing positions or vehicle movement modes, are built in your own modules on the framework's public seams.

See [Plugin Ecosystem](/ecosystem/) for the dependency map.

## 3. Author units as Blueprints

A unit type is a Blueprint subclass of `ASeinActor`. Add native or Blueprint-authored Sein components through the Components panel; its `USeinEntityBridgeComponent` bakes their authored values into the unit's simulation data at spawn. Players and the match work the same way: the Game Mode's Player State and Game State Blueprints carry the Sein components for each player and for the match.

Components remain pure data. Behavior belongs in abilities, effects, AI controllers, command brokers, and simulation systems.

## 4. Compose commands as abilities

Move, attack, harvest, build, patrol, garrison, and reinforce all use the same ability model. C++ supplies deterministic operations and lifecycle plumbing; Blueprint graphs compose the game-specific command flow.

Continue with [Units, Components, and Abilities](/core-concepts/units-components-abilities/).

## 5. Preserve the simulation boundary

Authoritative simulation code uses fixed-point types, entity handles, and the deterministic random generator. Unreal actors, floating-point vectors, and presentation-only systems stay outside simulation state.

Read [Deterministic Simulation](/core-concepts/deterministic-simulation/) before extending the simulation layer.

## Build the demo from a blank project

Start with [Create the Demo Project](/guides/creating-the-demo-project/). The walkthrough begins with plugin installation and project settings. You create the gameplay assets yourself.

The complete sequence builds a Soldier, movement and combat, mineral income, a HUD, Barracks and Factory production, a research unlock, fog and minimap, and a two-player lobby. It ends with [packaging and gameplay verification](/guides/packaging-and-verifying-the-demo/). The gameplay graphs are authored in Blueprint.
