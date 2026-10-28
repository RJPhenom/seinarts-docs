---
title: Units, Components, and Abilities
description: How SeinARTS turns Blueprint-authored units, players and the match into deterministic simulation data.
---

SeinARTS keeps designer-facing authoring in Blueprint while the runtime simulation operates on deterministic data.

## The Blueprint is the unit

A unit type is a Blueprint subclass of `ASeinActor`. The actor automatically owns a `USeinEntityBridgeComponent`, which connects its Unreal presentation to its simulation entity.

Designers add native or Blueprint-authored Sein components through the unit Blueprint's Components panel. These derive from `USeinActorComponent`. The bridge bakes their authored values into its `ComponentData` array as deterministic data stored in `FInstancedStruct` entries.

At spawn, the simulation copies the authored data into its component storage. After that boundary, simulation systems work with entity handles and component data rather than actors.

## Players and the match follow the same rule

The Game Mode's **Player State Class** and **Game State Class** are Blueprints too. Their Sein components define each player's entity and the match's entity in the simulation: resources and custom player data on the Player State, match-wide data on the Game State. Passive abilities on them run economy rules such as income, or match rules such as victory conditions. A victory rule calls **Defeat Player** or **End Match**; once one team remains the match ends on its own. A defeated player whose team plays on stays to watch: they see what a teammate still playing sees (the whole map once none is left), can give no orders, and lose their units unless Basic Match Settings keeps them. The Player Controller's **On Defeated** is where a defeat screen goes. Every player's simulation stops on the same tick, the Player Controller's **On Match Ended** shows each player its result, and the Game Mode's **On Match Ended** decides what comes next (by default, a return to the lobby).

The Player Controller and AI Controller hold no simulation data. They submit commands, the same way for a human and a computer player.

## Components are data

Components describe state. They do not own event graphs or state-changing behavior.

Framework simulation structs use the `SeinDeterministic` metadata marker so the editor can accept them as component data and validate their fields. Designers create their own deterministic components with the SeinARTS Actor Component asset.

Logic belongs in:

- Abilities.
- Effects.
- AI controllers.
- Command brokers.
- Simulation systems.

## Effects change component values

An effect's **Modifiers** change fields of components or abilities while it is active: Speed on Movement Data, Max Range and Cooldown on an ability, Damage on your own Combat component, a resource cap on the Player State. A modifier names a field by path (`VisionStamps[0].Radius`, `ResourceCaps[SeinARTS.Resource.Minerals]`) and either adds to a number, scales it (percentages add, so two +30% make +60%), or overrides any type of value, such as a weapon class or a bool.

The field itself holds the modified value, so movement, production, abilities, Blueprint reads and the UI all use it without asking. The value without modifiers, its base, is kept beside it: read it with **Get Base Data** or **Get Attribute**, and read what a class would start with for a player, before any unit exists, with **Get Class Data**. Writing a modified field sets its base, and the modifiers apply on top again.

An effect's **Scope** decides who it reaches: the unit it was applied to, every unit of a class the player owns (including ones built later), or the player itself. **Instant** effects change a base for good; **Persistent** and **Timed** ones last while they are active. A unit's **On Attributes Changed** event lets its Blueprint react, for example by swapping a weapon model.

## Everything is an ability

Gameplay commands share one extensible model. Moving, attacking, harvesting, building, patrolling, garrisoning, and reinforcing are ability Blueprints rather than values in a hard-coded command enum.

C++ provides deterministic primitives, command activation and cancellation, and latent execution infrastructure. Designers compose those pieces into Blueprint graphs for the needs of their game. An ability moves its unit with **Move To** and turns a standing unit with **Face Point**, which holds until the unit next moves: an attack faces its target this way.

## Presentation reacts to state

The actor and visual layer read the simulation. Animation, effects, audio, and other presentation stay downstream and cannot write back into authoritative state.

Abilities tell presentation what happened with the Notify nodes. **Notify Event** sends a game-defined event, such as a shot, to the Primary Entity's actor, where the Entity Bridge's **On Visual Event** plays its effect. Every machine plays it from the same event, and the simulation never reads it.
