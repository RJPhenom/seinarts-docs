---
title: Add Combat
description: Author health and weapon data as Sein components, an Attack order with a fixed-time firing loop, a passive Auto-Attack, and a muzzle flash and tracer for every shot.
---

![Three Soldiers trading fire with an attacking group; the health bars and the Truck come from later chapters.](../../../assets/guides/adding-combat/combat-header.png)

Complete [Add Movement](/guides/adding-movement/) first. This chapter gives the Soldier health and a rifle: 100 health, 10 damage per shot, 1.5 shots a second, 900 units of range. It attacks what you right-click and fires on its own at anything hostile nearby.

The framework has no health or weapon system. Combat is game data and abilities that you author; the framework supplies the pieces they use: component storage, targeting queries, field writes and death notifications.

## 1. Health and weapon components

Create the folder `Content/Demo/Components`. Right-click and choose **SeinARTS > SeinARTS Actor Component** for each:

| Component | Variables (Fixed Point, Instance Editable) |
| --- | --- |
| `SC_Health` | `MaxHealth` `100`, `Health` `100` |
| `SC_Combat` | `Damage` `10`, `RateOfFire` `1.5`, `Range` `900` |

A Sein component is simulation data: compiling it generates a data struct (`SC_HealthData`, `SC_CombatData`) that the simulation stores per entity, saves and checks across players. Use Fixed Point, never float, for anything gameplay reads.

Add both to `SU_Soldier`, compile and save.

## 2. The Attack order

Create an ability `SA_Attack` (**SeinARTS > SeinARTS Ability**, Generic Ability):

| Setting | Value |
| --- | --- |
| Ability Name | `Attack` |
| Target Type | Entity |
| Max Range | `900` |
| Requires Line Of Sight | On |
| Out Of Range Behavior | Auto Move Then |
| Granted Tags | `SeinARTS.Ability.Attack` |
| Cancel Abilities With Tag | `SeinARTS.Ability` |

**Auto Move Then** walks the unit into range with its Move Ability before activating. Add a Fixed Point variable `TimeUntilShot`.

### One shot: TryShot

Add a function `TryShot` with an input `Target` (Sein Entity Handle) and a Boolean output `Continue`:

1. Read the owner's weapon with **Get Attribute** (Entity **Owner Entity**, Component Type `SC_CombatData`, Field `Damage`, and again for `Range`). Get Attribute reads the field's effective value, after any upgrades.
2. **Make Sein Target Query**: Instigator **Owner Entity**, Range from `Range`, Required Component `SC_HealthData`. **Check Target** with the query and `Target`, then **Switch on ESeinTargetCheckResult**. Every result but **Eligible** returns `Continue` false.
3. On Eligible: add a local Fixed Point variable `Dealt` and set it to the **Min** of Damage and the target's `Health`, the damage this shot actually deals. Then **Apply Field Delta** on the target's `SC_HealthData` field `Health` with Delta **Negate** Damage and **Clamp Min** on, and after it **Notify Damage Applied** (Target, Source **Owner Entity**, Amount `Dealt`). `Dealt` is read before the write, so a killing shot still reports the health it took. Then **Notify Event**: Event Tag `SeinARTS.Event.Fire` (add the tag when prompted), Primary Entity **Owner Entity**, Secondary Entity `Target`; expand the node's advanced pins and set Location to the Location of **Get Entity Transform** on `Target` (**Break Fixed Transform**).
4. If **At Min**, the target is dead: **Notify Death** (Dying the target, Killer the owner), **Destroy Entity** on the target, and return false. Otherwise return true.

![SA_Attack's TryShot.](../../../assets/guides/adding-combat/tryshot-graph.png)

Apply Field Delta writes simulation state, so every player's simulation applies the same hit. The Notify nodes only tell presentation what happened: the hit, the shot and the death. Notify Event carries a game-defined event, here `SeinARTS.Event.Fire`, to the Primary Entity's actor; section 4 turns it into a visible shot.

### The firing loop

In the Event Graph:

- **On Activate**: set `TimeUntilShot` to **Zero**, so the first shot comes at once.
- **On Tick**: subtract **Delta Time** from `TimeUntilShot`. When it is at or below zero, call `TryShot` with **Target Entity**. If it returns true, add **One** divided by the owner's `RateOfFire` to `TimeUntilShot`; otherwise **End Ability**.

![SA_Attack's firing loop.](../../../assets/guides/adding-combat/attack-graph.png)

On Tick runs at the simulation's fixed rate and Delta Time is fixed point, so every player fires on the same tick. Never use Delay, timers or Event Tick for gameplay.

## 3. Auto-Attack

A unit standing idle should defend itself. Right-click `SA_Attack` and choose **Create Child Blueprint Class**, named `SA_AutoAttack`. It inherits `TryShot` and `TimeUntilShot`. In its Class Defaults:

| Setting | Value |
| --- | --- |
| Ability Name | `Auto-Attack` |
| Is Passive | On |
| Target Type | None |
| Max Range | `0` |
| Requires Line Of Sight | Off |
| Out Of Range Behavior | Reject |
| Granted Tags | Empty |
| Cancel Abilities With Tag | Empty |

A passive ability activates when granted and ticks for the unit's lifetime. With no Granted Tags, no order cancels it, and Stop leaves it running.

Override **On Tick**:

1. Skip the tick when the owner **Has Tag** `SeinARTS.Ability`, meaning an order is running, or **Is In World** is false, for example while riding in a Truck.
2. Count `TimeUntilShot` down to zero, clamping with **Max** so it waits at zero.
3. At zero, **Find Targets** with the same query as `TryShot`. If the result **Is Not Empty**, call `TryShot` on the first candidate's **Target** and set `TimeUntilShot` to **One** divided by `RateOfFire`.

![SA_AutoAttack's On Tick.](../../../assets/guides/adding-combat/autoattack-graph.png)

Find Targets returns the nearest eligible entities first. The default scorer skips only entities the unit's own player owns, and Required Component skips anything without health, such as mineral rocks. In a game with allied players, set the query's **Scorer Class** to a target scorer that also skips allies, or Auto-Attack fires on them.

## 4. Show the shot

Each shot now sends `SeinARTS.Event.Fire` with the shooter, the target and the target's position. The shooter's actor turns it into a muzzle flash and a tracer.

### The effect

Create the folder `Content/Demo/Art/Effects`. Right-click it and choose **FX > Niagara System**. In **Create Niagara System**, pick the **Static Beam** emitter under **Templates**, create the system and name it `NS_RifleShot`. Open it, right-click the System Overview, choose **Add Emitter** and add **Simple Sprite Burst**. Rename the two emitters `Tracer` and `MuzzleFlash`.

In the **User Parameters** panel, add a **Vector** parameter named `BeamEnd`: the world position the tracer ends at. Then set:

| Emitter | Module | Input | Value |
| --- | --- | --- | --- |
| Tracer | Emitter State | Loop Behavior | Once |
| Tracer | Emitter State | Loop Duration | `0.1` |
| Tracer | Beam Emitter Setup | Beam End | `User.BeamEnd`, from **Link Input** in the input's dropdown |
| Tracer | Beam Emitter Setup | Absolute Beam End | On |
| Tracer | Spawn Burst Instantaneous | Spawn Count | `20` |
| Tracer | Initialize Particle | Lifetime | `0.08` |
| Tracer | Beam Width | Beam Width | `3` (reset the input with its arrow first, which replaces the template's curve with a plain value) |
| Tracer | Color | Color | R `6`, G `4`, B `1.5`, A `1` |
| MuzzleFlash | Emitter State | Loop Duration | `0.1` |
| MuzzleFlash | Initialize Particle | Lifetime | `0.06` |
| MuzzleFlash | Initialize Particle | Color | R `10`, G `7`, B `3`, A `1` |
| MuzzleFlash | Initialize Particle | Uniform Sprite Size | `35` |

The beam starts where the system is spawned and ends at `BeamEnd`; both emitters play once and the system finishes after a tenth of a second. Colour values above 1 glow.

### Play it on the Soldier

In `SU_Soldier`, select **SeinARTS Entity Bridge** in the Components panel and, under **Events** in its Details, click **+** next to **On Visual Event**. The bridge raises it for every visual event on the Soldier's entity, and Notify Event sends `SeinARTS.Event.Fire` to the shooter. From the event:

1. **Break Sein Visual Event**, and **Branch** on its Tag **==** `SeinARTS.Event.Fire`.
2. True: **Spawn System Attached**: System Template `NS_RifleShot`, Attach to Component **Default Scene Root**, Location the muzzle, Location Type **Keep World Position**, Auto Destroy on. For the muzzle, **Transform Location** of the `Rifle`'s **Get World Transform** with Location `(380, 0, 10)`: the barrel's tip in the demo's rifle model; use your model's.
3. **Set Niagara Variable (Vector3)** on the spawned component: In Variable Name `BeamEnd`, In Value the event's Location through **To Vector**, plus `(0, 0, 90)` to aim at the target's body.

![SU_Soldier's On Visual Event.](../../../assets/guides/adding-combat/fire-effect-graph.png)

The effect is presentation only: each player's machine plays it from the event, and the simulation never reads it. It is attached to the Soldier, so it hides with the Soldier in fog.

## 5. Grant and map

On `SU_Soldier`'s Sein Abilities, add `SA_Attack` and `SA_AutoAttack` to Granted Abilities, and a Default Command:

| Required Context | Ability Tag | Priority |
| --- | --- | --- |
| Right Click, Target Enemy | `SeinARTS.Ability.Attack` | `10` |

Compile and save.

## 6. Test

Set the slot 2 start's **Spawn Entity** to `SU_Soldier` as well, and play. Walk your Soldier towards the other start and right-click the enemy Soldier: yours closes to `900` units and fires about every 0.67 seconds, each shot a flash at the muzzle and a tracer to the target, until one of them dies, ten hits later. Then order a fresh Soldier near the enemy without targeting it: once it stops, Auto-Attack opens fire.

## Checkpoint

- `SC_Health` and `SC_Combat` are Fixed Point simulation data on the Soldier.
- `SA_Attack` checks its target and writes Health with Apply Field Delta on a fixed-time loop.
- `SA_AutoAttack` fires only while the Soldier has no order.
- Each shot sends `SeinARTS.Event.Fire`, and the Soldier plays `NS_RifleShot` from its On Visual Event.

Continue with [Add Mineral Income](/guides/adding-mineral-income/).
