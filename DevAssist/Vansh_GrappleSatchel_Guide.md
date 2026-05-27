# PROJECT ANOMALY — VANSH'S IMPLEMENTATION GUIDE
## Grapple Hook + Satchel Charge · UE 5.4.4 · Branch: Vansh

> **ADHD-FRIENDLY FORMAT:** Every section starts with a 1-sentence summary of what you are doing and why.
> Callouts marked **⚠️ STOP** mean stop and check before continuing.
> Callouts marked **✅ BEFORE YOU START** are prerequisites — nothing will work without them.
> Callouts marked **🔧 DEBUG** tell you how to test each thing after you build it.

---

## QUICK NAVIGATION

| I want to... | Go to |
|---|---|
| Understand the full picture first | [Section 1 — What You're Building](#section-1--what-youre-building) |
| See what my teammate is doing | [Section 2 — Teammate Split](#section-2--teammate-split--what-not-to-touch) |
| Know what I need before starting | [Section 3 — Prerequisites](#section-3--prerequisites-build-these-first) |
| Start building the Grapple Hook | [Section 4 — BP_GrappleAnchor](#section-4--bp_grappleanchor) |
| Build the movement component | [Section 5 — AC_MovementExtComponent](#section-5--ac_movementextcomponent) |
| Build the Satchel Charge actor | [Section 6 — BP_SatchelCharge](#section-6--bp_satchelcharge) |
| Build the utility component | [Section 7 — AC_UtilityComponent](#section-7--ac_utilitycomponent) |
| Wire inputs in the player character | [Section 8 — Input Wiring](#section-8--wiring-inputs-in-bp_playercharacter) |
| Test everything works | [Section 9 — Debug & Test Checklist](#section-9--debug--test-checklist) |
| Prepare for GitHub merge | [Section 10 — GitHub Handoff](#section-10--github-handoff) |
| Look up terminology | [Section 11 — Glossary](#section-11--glossary) |

---

## SECTION 1 — WHAT YOU'RE BUILDING

**You are building two complete systems that handle the player's movement toolkit (grapple) and tactical ability (satchel).**

### The Grapple Hook
The player presses **E** and fires a line trace from the camera forward. If it hits a **BP_GrappleAnchor** actor placed in the level, the player is launched through the air toward that anchor point. It works at all times — no rage requirement.

In UE terms, this is a **Line Trace** → **Cast to BP_GrappleAnchor** → **Launch Character** chain, managed by a component called **AC_MovementExtComponent**.

### The Satchel Charge
The player presses **Q** to throw a small physics object (BP_SatchelCharge) that bounces off surfaces. After 2 seconds, it detonates — dealing damage in a radius and applying a radial force that can blast-jump the player. It has a 45-second cooldown. Managed by **AC_UtilityComponent**.

### Your 5 Deliverables

| Asset | Type | Folder | Purpose |
|---|---|---|---|
| `BP_GrappleAnchor` | Blueprint Actor | `Content/ProjectAnomaly/Arena/` | Level-placed target marker for grapple |
| `AC_MovementExtComponent` | Actor Component | `Content/ProjectAnomaly/Player/Components/` | Owns TryGrapple() logic |
| `BP_SatchelCharge` | Blueprint Actor | `Content/ProjectAnomaly/Combat/` | Physics projectile with fuse + detonation |
| `AC_UtilityComponent` | Actor Component | `Content/ProjectAnomaly/Player/Components/` | Owns ThrowSatchel() + cooldown |
| *(Wiring)* | BP_PlayerCharacter | *(teammate's file — add only, don't delete)* | Connects your components via input events |

---

## SECTION 2 — TEAMMATE SPLIT & WHAT NOT TO TOUCH

**Your teammate owns specific files. If you open and save them, you will create a merge conflict in GitHub.**

### What Your Teammate Is Building

| System | Files They Own | Why You Must Not Touch |
|---|---|---|
| Basic movement | `BP_PlayerCharacter`, `BP_PlayerController` | They are wiring movement input (WASD, jump, look) |
| Rage system | `AC_RageComponent` | They own the rage meter logic |
| Rage-gated Dash | The `TryDash()` function inside `AC_MovementExtComponent` | They will add this function after merge |

### The AC_MovementExtComponent Split

This is the trickiest part. Both you and your teammate share this one component:
- **You build:** `TryGrapple()` and all grapple-related variables
- **Your teammate adds later:** `TryDash()` and dash variables

**Strategy:** You create `AC_MovementExtComponent` and stub out a clearly-labelled empty Custom Event called `TryDash` with a comment. Your teammate fills it in after the merge.

### What You CAN Touch Safely

You are the sole author of these files — no merge conflicts possible:
- `BP_GrappleAnchor` (you create this fresh)
- `AC_MovementExtComponent` (you create this fresh)
- `BP_SatchelCharge` (you create this fresh)
- `AC_UtilityComponent` (you create this fresh)

For `BP_PlayerCharacter`, you will **add** the input event bindings for IA_Grapple and IA_Utility. Coordinate with your teammate so you are not both editing the same BeginPlay sequence at the same time. Best practice: **finish your components first, then both of you edit BP_PlayerCharacter together at one workstation.**

---

## SECTION 3 — PREREQUISITES (BUILD THESE FIRST)

**These assets must exist before you start. If any are missing, stop and build them first.**

### Layer 0 — Must Exist Before Anything

These have zero dependencies. If your teammate hasn't built them yet, you can build them yourself — they are pure data assets with no Blueprint graphs.

| Asset | Location | How to Create | Why You Need It |
|---|---|---|---|
| `FS_DamageInfo` | `Content/ProjectAnomaly/Data/Structs/` | Right-click → Blueprint → Structure | Satchel uses this struct to deal damage |
| `BPI_Damageable` | `Content/ProjectAnomaly/Interfaces/` | Right-click → Blueprint → Blueprint Interface | Satchel calls ReceiveDamage through this interface |
| `IA_Grapple` | `Content/ProjectAnomaly/Input/` | Right-click → Input → Input Action | Grapple input trigger |
| `IA_Utility` | `Content/ProjectAnomaly/Input/` | Right-click → Input → Input Action | Satchel throw input trigger |
| `IMC_Default` | `Content/ProjectAnomaly/Input/` | Right-click → Input → Input Mapping Context | Binds keys to actions |

> **⚠️ STOP — Check FS_DamageInfo fields.**
> If it already exists, open it and confirm it has exactly these 6 fields or the satchel damage calls will break at compile time:
> - `BaseDamage` (float)
> - `bIsWeakPointHit` (bool)
> - `bIsArmourDamage` (bool)
> - `bIsNextShotBoosted` (bool)
> - `Instigator` (Object Reference → Actor)
> - `HitLocation` (Vector)

### Layer 0 — FS_DamageInfo Setup (Only If It Doesn't Exist)

1. Navigate to `Content/ProjectAnomaly/Data/Structs/`
2. Right-click → **Blueprint → Structure** → name it `FS_DamageInfo`
3. Open it → click the **+** button 6 times → add each field listed above
4. Field names are **case-sensitive** — typos break every damage call silently
5. Compile and Save

### Layer 0 — BPI_Damageable Setup (Only If It Doesn't Exist)

1. Navigate to `Content/ProjectAnomaly/Interfaces/`
2. Right-click → **Blueprint → Blueprint Interface** → name it `BPI_Damageable`
3. Open it → add these functions using the **+** button in the My Blueprint panel:
   - `ReceiveDamage` — Input: `DamageInfo` (FS_DamageInfo), No return value
   - `IsAlive` — No inputs, Return: bool
   - `GetCurrentHealth` — No inputs, Return: float
4. Save

### Layer 0 — Input Actions Setup (Only If They Don't Exist)

For `IA_Grapple` and `IA_Utility`:
1. Navigate to `Content/ProjectAnomaly/Input/`
2. Right-click → **Input → Input Action**
3. Name it `IA_Grapple`, open it → **Value Type = Digital (bool)**
4. Repeat for `IA_Utility` → **Value Type = Digital (bool)**
5. Open `IMC_Default` → click **+** → assign `IA_Grapple` → bind it to **E**
6. Click **+** again → assign `IA_Utility` → bind it to **Q**
7. Save

> **Official UE5.4 Docs Reference:** Enhanced Input System
> https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine

---

## SECTION 4 — BP_GrappleAnchor

**This is the simplest asset you'll build — it is a pure target marker with no Blueprint logic.**

### What It Is

A small sphere collision actor placed around the arena at height-varied positions. The grapple's line trace checks if it hit one of these. If yes, the player launches toward it. If no, nothing happens.

### Folder

```
Content/ProjectAnomaly/Arena/
```

### Step-by-Step Build

#### Step 1 — Create the Blueprint

1. In the **Content Browser**, navigate to `Content/ProjectAnomaly/Arena/`
   - If this folder doesn't exist: Right-click in Content Browser → **New Folder** → name it `Arena`
2. Right-click inside `Arena/` → **Blueprint Class**
3. In the class picker, search for `Actor` → select it → click **Select**
4. Name it exactly: `BP_GrappleAnchor`
5. Double-click to open it

#### Step 2 — Add a Sphere Collision Component

1. In the **Components** panel (top-left), click **+ Add**
2. Search for `Sphere Collision` → select it
3. With the Sphere Collision selected, go to the **Details** panel:
   - **Sphere Radius:** `50.0`
   - **Collision Presets:** `OverlapAllDynamic` (this ensures line traces can detect it)
4. Rename the component to `GrappleSphere`

#### Step 3 — Add a Visible Mesh (Editor Only)

This makes the anchor visible in the editor viewport so you know where you placed them.

1. Click **+ Add** again → search for `Static Mesh` → select it
2. Name it `AnchorMesh`
3. In Details → **Static Mesh** → pick any small sphere mesh (e.g. `SM_Sphere` from engine content)
4. Scale it down: **X=0.3, Y=0.3, Z=0.3** so it's a small visual marker

> **There is NO Blueprint graph needed.** This actor has zero nodes. It is only a position marker. The grapple logic lives in AC_MovementExtComponent.

#### Step 4 — Compile and Save

Click **Compile** (top-left, green arrow) → click **Save**.

#### Step 5 — Place Instances in the Level

1. Go to your arena level in the editor
2. Drag `BP_GrappleAnchor` from the Content Browser into the viewport
3. Place **at least 3 anchors** at different heights:
   - One low (reachable from ground, 200–300 units up)
   - One mid (400–600 units up)
   - One high (800+ units up, reachable only after a jump or lower grapple)
4. Make sure none are inside walls or unreachable geometry

> **🔧 DEBUG:** Press **P** in Play In Editor (PIE) to see the NavMesh. Anchors don't need NavMesh but this helps you see the arena layout to pick good positions.

> **Official UE5.4 Docs Reference:** Blueprint Actor classes
> https://dev.epicgames.com/documentation/en-us/unreal-engine/blueprint-class-assets-in-unreal-engine

---

## SECTION 5 — AC_MovementExtComponent

**This component owns the TryGrapple() function. You build TryGrapple fully. Leave TryDash as an empty stub for your teammate.**

### What It Does

- **TryGrapple():** Fires a line trace from the player's camera. If it hits a `BP_GrappleAnchor`, stores that anchor as `CurrentAnchor` and calls `Launch Character` on the owning player toward the anchor at `GrappleSpeed`.
- **TryDash() (STUB):** Empty Custom Event that your teammate will implement after the merge.

### Folder

```
Content/ProjectAnomaly/Player/Components/
```

### Variables Reference Table

| Variable Name | Type | Default | Purpose |
|---|---|---|---|
| `CurrentAnchor` | Object Reference (BP_GrappleAnchor) | None | Stores last grappled anchor |
| `GrappleSpeed` | float | `2500.0` | How fast the player launches toward the anchor |
| `GrappleRange` | float | `3000.0` | Max distance a grapple line trace can reach |
| `bIsGrappling` | bool | false | True while the grapple pull is in flight |
| `bDashAvailable` | bool | false | Controlled by rage system — your teammate sets this |

### Event Dispatchers Reference Table

| Dispatcher Name | Payload | Who Binds to It |
|---|---|---|
| `OnGrappleAttached` | None | ABP_Player (for animation state) |
| `OnDashUsed` | None | ABP_Player (for animation state) |

### Step-by-Step Build

#### Step 1 — Create the Component

1. Navigate to `Content/ProjectAnomaly/Player/Components/`
   - If the folder doesn't exist: Right-click → **New Folder** → name it `Components`
2. Right-click inside → **Blueprint Class**
3. In the class picker, search for `ActorComponent` → select it
4. Name it exactly: `AC_MovementExtComponent`
5. Double-click to open it

#### Step 2 — Add Variables

Open the **My Blueprint** panel (left side). Under **Variables**, click **+** for each:

| Click + | Variable Name | Type | Default Value | Instance Editable? |
|---|---|---|---|---|
| + | `CurrentAnchor` | Object Reference → `BP_GrappleAnchor` | None | No |
| + | `GrappleSpeed` | Float | `2500.0` | Yes |
| + | `GrappleRange` | Float | `3000.0` | Yes |
| + | `bIsGrappling` | Boolean | false | No |
| + | `bDashAvailable` | Boolean | false | No |

> **How to set Instance Editable:** Click the variable → Details panel → check the **Instance Editable** box (eye icon in some versions).

#### Step 3 — Add Event Dispatchers

In **My Blueprint** → **Event Dispatchers** section → click **+** twice:
1. Name the first: `OnGrappleAttached`
2. Name the second: `OnDashUsed`

No inputs needed for either dispatcher.

#### Step 4 — Build the TryGrapple Function

In My Blueprint → **Functions** → click **+** → name it `TryGrapple`.

Inside the TryGrapple function graph, build this node sequence:

```
[TryGrapple Entry]
        │
        ▼
[Get Owner]  ──►  [Cast to Character]
                          │
                          ▼
                 [Get Actor Location]  ──►  stored as "PlayerLoc"
                 [Get Controller] ──► [Cast to PlayerController]
                          │
                          ▼
                 [Get Player View Point]
                 (outputs: Location "CamLoc", Rotation "CamRot")
                          │
                          ▼
                 [Get Forward Vector from CamRot]  ──► "CamFwd"
                          │
                          ▼
              [LineTraceByChannel]
                 Start: CamLoc
                 End: CamLoc + (CamFwd × GrappleRange)
                 TraceChannel: Visibility
                 bTraceComplex: false
                          │
                    ┌─────┴──────┐
             bBlockingHit?   bBlockingHit?
               TRUE             FALSE
                │                 │
                ▼                 ▼
     [Break HitResult]       [Return] (exit silently)
         │
         ▼
   [Cast HitActor to BP_GrappleAnchor]
         │
     ┌───┴────┐
   SUCCESS   FAIL
     │         │
     ▼         ▼
[SET CurrentAnchor    [Return] (exit silently —
 = CastResult]         not a valid anchor)
     │
     ▼
[GET CurrentAnchor → GetActorLocation] → "AnchorLoc"
     │
     ▼
[AnchorLoc - PlayerLoc]  ──►  [Normalize Vector]  ──►  "LaunchDir"
     │
     ▼
[LaunchDir × GrappleSpeed]  ──►  "LaunchVelocity"
     │
     ▼
[Get Owner] → [Cast to Character]  ──►  [Launch Character]
                 Launch Velocity: LaunchVelocity
                 bXYOverride: TRUE
                 bZOverride: TRUE
     │
     ▼
[SET bIsGrappling = TRUE]
     │
     ▼
[Call OnGrappleAttached dispatcher]
     │
     ▼
[Return]
```

**Step-by-step in the Blueprint editor:**

1. Right-click in the graph → search `Get Owner` → add it
2. Drag the output pin → search `Cast to Character` → add it → connect
3. From Cast's `As Character` pin → drag → search `Get Controller` → add it
4. From the Controller pin → drag → search `Cast to PlayerController` → add it
5. From `As Player Controller` → drag → search `Get Player View Point` → add it
   - This gives you **Location** and **Rotation** output pins
6. From **Rotation** output → drag → search `Get Forward Vector` → add it (gives you camera forward direction)
7. Right-click in graph → search `Line Trace By Channel` → add it
   - **Start:** Connect `Location` from Get Player View Point
   - **End:** You need `Location + (ForwardVector × GrappleRange)`. To build this:
     - Right-click → search `float * vector` → add a **Float × Vector** multiply node
     - Connect `GrappleRange` variable to the float pin
     - Connect `Get Forward Vector` output to the vector pin
     - Right-click → search `vector + vector` → add an **Add Vectors** node
     - Connect `Location` and the multiply result to the add node
     - Connect the add result to the **End** pin
   - **Trace Channel:** `Visibility`
   - Leave all other settings default
8. From the `Line Trace` → drag the **Out Hit** pin → search `Break Hit Result` → add it
9. From `Break Hit Result` → drag the **Hit Actor** pin → search `Cast to BP_GrappleAnchor` → add it
10. Wire the `bBlockingHit` output through a **Branch** node:
    - Right-click → `Branch` → connect `bBlockingHit` to the **Condition** pin
    - **True** exec pin → connect to the `Cast to BP_GrappleAnchor` node
    - **False** exec pin → connect to a **Return Node** (right-click → `Return Node`)
11. From `Cast to BP_GrappleAnchor` → drag the **Cast Failed** exec pin → connect to another **Return Node**
12. From `Cast to BP_GrappleAnchor` → `As BP_GrappleAnchor` → drag → `Get Actor Location` → stores the anchor position
13. From `Cast to BP_GrappleAnchor` success exec pin → **SET CurrentAnchor** (drag from variable)
    - Connect `As BP_GrappleAnchor` to the SET pin
14. After SET CurrentAnchor → compute launch direction:
    - Right-click → `Vector - Vector` → subtract `Get Owner → Get Actor Location` from anchor location
    - Right-click → `Normalize Vector` → connect the subtraction result
    - Right-click → `Vector × Float` → multiply normalized dir by `GrappleSpeed` variable
15. From there → `Get Owner → Cast to Character → Launch Character`:
    - **Launch Velocity:** the multiplied vector
    - **bXYOverride:** checked (true)
    - **bZOverride:** checked (true)
16. After Launch Character → **SET bIsGrappling = true**
17. After SET → **Call OnGrappleAttached** (drag the dispatcher from My Blueprint → Call)

> **Official UE5.4 Docs Reference:** Line Traces
> https://dev.epicgames.com/documentation/en-us/unreal-engine/traces-in-unreal-engine---overview
>
> **Official UE5.4 Docs Reference:** Launch Character
> https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/GameFramework/ACharacter/LaunchCharacter

#### Step 5 — Build the TryDash Stub

This is for your teammate. Create a placeholder so the component compiles cleanly.

1. In My Blueprint → **Functions** → click **+** → name it `TryDash`
2. Add an input parameter: `Direction` (Vector)
3. Inside the function graph, **do not add any nodes** — just the entry point
4. Compile → it should compile clean (empty function is valid)

> **For your teammate:** This is where you add the dash logic after the merge. Wire `Launch Character` here using the `bDashAvailable` variable as the gate.

#### Step 6 — Compile and Save

Click **Compile** → should show a green checkmark → click **Save**.

> **🔧 DEBUG — Test Grapple Compiles:**
> After compile, if you see red errors mentioning `BP_GrappleAnchor`, make sure that asset exists in `Content/ProjectAnomaly/Arena/` first. Open it, compile it, then re-open AC_MovementExtComponent and compile again.

---

## SECTION 6 — BP_SatchelCharge

**This is the physical grenade actor. It handles its own bounce physics, 2-second fuse, detonation damage, and radial shockwave.**

### What It Does (Step by Step at Runtime)

1. AC_UtilityComponent spawns it at the player's hand socket position
2. An impulse is added in the forward direction — it flies through the air as a physics actor
3. A 2-second timer starts (the "fuse")
4. Timer fires → `Detonate` event runs:
   - `RadialForceComponent` fires its impulse (pushes nearby actors including the player for blast jump)
   - Sphere Overlap check → all overlapping actors receive `BPI_Damageable → ReceiveDamage`
   - Niagara explosion spawns (placeholder for now if NS_SatchelExplosion doesn't exist yet)
   - Actor destroys itself

### Folder

```
Content/ProjectAnomaly/Combat/
```

### Components Needed

| Component | Settings |
|---|---|
| Static Mesh | Small sphere shape, for visuals |
| Sphere Collision | Radius: `15.0`, Collision: `PhysicsActor` |
| RadialForceComponent | Radius: `600.0`, Strength: `200000.0`, bImpulseVelChange: `true`, bIgnoreOwningActor: `false` |

### Variables Reference Table

| Variable | Type | Default | Purpose |
|---|---|---|---|
| `BaseDamage` | Float | `100.0` | Damage per overlapping actor at detonation |
| `BlastRadius` | Float | `600.0` | Sphere overlap radius for damage at detonate |
| `FuseDuration` | Float | `2.0` | Seconds until detonation |
| `InstigatorRef` | Object Reference (Actor) | None | Set by AC_UtilityComponent after spawn |

### Step-by-Step Build

#### Step 1 — Create the Blueprint

1. Navigate to `Content/ProjectAnomaly/Combat/`
   - If it doesn't exist: Right-click → **New Folder** → name `Combat`
2. Right-click → **Blueprint Class** → pick `Actor` → name it `BP_SatchelCharge`
3. Double-click to open

#### Step 2 — Add Components

In the **Components** panel:

**Sphere Collision (Root):**
1. Click **+ Add** → `Sphere Collision` → rename to `SatchelCollision`
2. Details panel:
   - **Sphere Radius:** `15.0`
   - **Collision Presets:** `PhysicsActor`
   - **Simulate Physics:** `true` (check this box)
   - **Enable Gravity:** `true` (check this box)

**Static Mesh:**
1. Click **+ Add** → `Static Mesh` → rename to `SatchelMesh`
2. Assign any small sphere mesh from engine content
3. Scale: `X=0.15, Y=0.15, Z=0.15`

**Radial Force:**
1. Click **+ Add** → `Radial Force` → rename to `BlastForce`
2. Details panel:
   - **Radius:** `600.0`
   - **Strength:** `200000.0`
   - **Impulse Strength:** `200000.0`
   - **bImpulseVelChange:** `true` ← **CRITICAL — blast jump won't work without this**
   - **bIgnoreOwningActor:** `false` ← So it also pushes the player

> **What bImpulseVelChange does:** When true, the force is applied as a velocity change rather than a force over time. This means heavy objects like the player character get flung just as far as light objects. Without it the player barely moves.

> **Official UE5.4 Docs Reference:** Radial Force Component
> https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/URadialForceComponent

#### Step 3 — Add Variables

In **My Blueprint → Variables**, click **+** for each:

| Variable Name | Type | Default |
|---|---|---|
| `BaseDamage` | Float | `100.0` |
| `BlastRadius` | Float | `600.0` |
| `FuseDuration` | Float | `2.0` |
| `InstigatorRef` | Object Reference → Actor | None |
| `FuseTimerHandle` | Timer Handle | (leave default) |

#### Step 4 — Build BeginPlay (Start the Fuse)

1. In the Event Graph, find the **Event BeginPlay** node
2. From BeginPlay → right-click → `Set Timer by Function Name`
   - **Object:** `Self`
   - **Function Name:** `Detonate` (type this exactly — this is the event you will create)
   - **Time:** Connect to `FuseDuration` variable
   - **Looping:** `false`
3. From `Set Timer by Function Name` → `Return Value` → connect to **SET FuseTimerHandle** variable
   - This stores the handle in case you ever need to clear the timer (e.g. satchel hits water)

#### Step 5 — Build the Detonate Event

1. Right-click in the graph → `Add Custom Event` → name it `Detonate`
2. From Detonate exec pin, build this sequence:

**Part A — Fire the Radial Force:**
1. Right-click → `Get BlastForce` (your Radial Force component reference)
2. Drag from it → `Fire Impulse`
3. Connect: `Detonate` exec → `Fire Impulse`

**Part B — Deal Damage in Radius:**
1. After `Fire Impulse` → right-click → `Get All Actors with Interface`
   - **Interface:** `BPI_Damageable`
   - This gets every actor in the world that implements the damageable interface
2. Drag the **Out Actors** pin → right-click → `For Each Loop`
3. Inside the loop body:
   - Right-click → `Get Self` (the satchel) → `Get Actor Location` → store as reference
   - Right-click → `Get Distance To` (from satchel to Array Element)
   - Branch: Is distance `<=` `BlastRadius`?
     - **True:** Continue to deal damage
     - **False:** Continue (skip)
4. When within radius → right-click → `Message (ReceiveDamage)` from BPI_Damageable interface:
   - **Target:** Array Element (the actor being iterated)
   - For the **DamageInfo** input, you need to `Make FS_DamageInfo` struct:
     - Right-click → `Make FS_DamageInfo`
     - **BaseDamage:** Connect `BaseDamage` variable
     - **bIsWeakPointHit:** false
     - **bIsArmourDamage:** false
     - **bIsNextShotBoosted:** false
     - **Instigator:** Connect `InstigatorRef` variable
     - **HitLocation:** Connect `Get Actor Location` of the satchel

**Part C — Spawn VFX (Placeholder):**
1. After the For Each loop completes → right-click → `Spawn System at Location`
   - **System Template:** Leave empty for now (assign `NS_SatchelExplosion` later when VFX are built)
   - **Location:** Satchel `Get Actor Location`

**Part D — Destroy Self:**
1. After VFX spawn → right-click → `Destroy Actor`
   - **Target:** `Self`

> **⚠️ STOP — Why "Get All Actors with Interface" and not "Get Overlapping Actors"?**
> `Get Overlapping Actors` only returns actors currently overlapping the collision component's capsule — which is tiny (radius 15). The blast radius is 600. You want to damage everything in 600 units. So you get all damageable actors and manually check distance. This is the correct approach.

> **Official UE5.4 Docs Reference:** Interfaces in Blueprints
> https://dev.epicgames.com/documentation/en-us/unreal-engine/implementing-blueprint-interfaces-in-unreal-engine

#### Step 6 — Compile and Save

Compile → green checkmark → Save.

> **🔧 DEBUG — Test Detonate:**
> Temporarily add a `Print String "DETONATE"` node right after the Detonate event entry. Place one BP_SatchelCharge in the level directly (not spawned by component). Hit **Play**. After 2 seconds, "DETONATE" should print. If it doesn't: check that the Function Name in Set Timer matches exactly "Detonate" (capital D).

---

## SECTION 7 — AC_UtilityComponent

**This component is the manager that throws the satchel, tracks cooldown, and tells the player when they can throw again.**

### What It Does

- `ThrowSatchel()`: Checks `bCanThrow` → spawns `BP_SatchelCharge` at the player's hand → gives it a forward impulse → starts the 45-second cooldown timer
- `StartCooldown` (Custom Event): Fires when the 45s timer expires → sets `bCanThrow = true` again

### Folder

```
Content/ProjectAnomaly/Player/Components/
```

### Variables Reference Table

| Variable | Type | Default | Instance Editable? | Purpose |
|---|---|---|---|---|
| `bCanThrow` | Boolean | true | No | Gates ThrowSatchel |
| `ThrowForce` | Float | `2000.0` | Yes | Impulse given to satchel on throw |
| `CooldownDuration` | Float | `45.0` | Yes | Seconds before can throw again |
| `CooldownTimerHandle` | Timer Handle | default | No | Stored to allow future cancellation |

### Event Dispatchers

| Dispatcher | Payload | Purpose |
|---|---|---|
| `OnSatchelThrown` | None | HUD can bind to show cooldown starting |
| `OnCooldownComplete` | None | HUD can bind to show ready indicator |

### Step-by-Step Build

#### Step 1 — Create the Component

1. Navigate to `Content/ProjectAnomaly/Player/Components/`
2. Right-click → **Blueprint Class** → `ActorComponent` → name it `AC_UtilityComponent`
3. Double-click to open

#### Step 2 — Add Variables and Dispatchers

Add all variables from the table above via **My Blueprint → Variables → +**.

Add dispatchers via **My Blueprint → Event Dispatchers → +**:
- `OnSatchelThrown`
- `OnCooldownComplete`

#### Step 3 — Build the ThrowSatchel Function

In My Blueprint → **Functions** → click **+** → name it `ThrowSatchel`.

Inside the function graph:

```
[ThrowSatchel Entry]
        │
        ▼
[Branch: bCanThrow?]
   TRUE         FALSE
    │              │
    ▼              ▼
[Continue]     [Return] (do nothing)
    │
    ▼
[Get Owner] → [Cast to Character]
    │
    ▼
[Get Mesh Component] (the character's skeletal mesh)
    │
    ▼
[Get Socket Location]
   Socket Name: "hand_r"   ← or whatever socket the player hand uses
    │
    ▼
[Get Socket Rotation]
   Socket Name: "hand_r"
    │
    ▼
[Spawn Actor from Class]
   Class: BP_SatchelCharge
   Location: Socket Location
   Rotation: Socket Rotation
   Collision Handling: AlwaysSpawn
    │
    ▼
[SET SpawnedSatchel.InstigatorRef = GetOwner()]
    │
    ▼
[GET SpawnedSatchel → Mesh Component → Add Impulse]
   Impulse: CameraForwardVector × ThrowForce
    │
    ▼
[SET bCanThrow = FALSE]
    │
    ▼
[Set Timer by Event: StartCooldown, Time: CooldownDuration, Looping: false]
   → Store in CooldownTimerHandle
    │
    ▼
[Call OnSatchelThrown dispatcher]
```

**Detailed node sequence:**

1. Right-click → `Branch` → Condition: `bCanThrow` variable
2. True exec → `Get Owner` → `Cast to Character`
3. From Character → `Get Mesh` (skeletal mesh component)
4. From Mesh → `Get Socket Location` → Socket Name: `"hand_r"` (check your skeleton for the correct socket name)
5. From Mesh → `Get Socket Rotation` → Socket Name: `"hand_r"`
6. Right-click → `Spawn Actor from Class`:
   - **Class:** `BP_SatchelCharge`
   - **Spawn Transform:** Compose from Socket Location + Socket Rotation (right-click → `Make Transform` → plug in Location and Rotation)
   - **Collision Handling Override:** `AlwaysSpawn`
7. Store the spawn return value → drag from it → `SET InstigatorRef` on the new satchel:
   - Right-click → `Get Owner` → connect to the InstigatorRef value
8. From the spawned satchel → right-click → `Get Static Mesh Component` (the SatchelMesh) → `Add Impulse`
   - For the impulse direction, get the camera forward:
     - `Get Owner` → `Cast to Character` → `Get Controller` → `Cast to PlayerController` → `Get Player View Point` → Rotation → `Get Forward Vector`
   - Multiply forward vector by `ThrowForce`
   - Connect to Add Impulse's **Impulse** pin
9. After impulse → **SET bCanThrow = false**
10. Right-click → `Set Timer by Event`:
    - **Event:** Right-click in the graph → `Add Custom Event` → name `StartCooldown` → connect to this field
    - **Time:** `CooldownDuration` variable
    - **Looping:** `false`
    - Store return in `CooldownTimerHandle`
11. Call `OnSatchelThrown` dispatcher

#### Step 4 — Build the StartCooldown Event

The `StartCooldown` Custom Event you created above should already be in the graph.

From its exec pin:
1. **SET bCanThrow = true**
2. **Call OnCooldownComplete** dispatcher

That's it. Two nodes.

#### Step 5 — Compile and Save

Compile → green checkmark → Save.

> **🔧 DEBUG — Test ThrowSatchel:**
> After wiring to input (Section 8), hit Play → press Q → you should see the BP_SatchelCharge spawn in the world, fly forward, and detonate after 2 seconds. If satchel spawns but doesn't fly: check the Add Impulse node — impulse value may be too low or the wrong component is being targeted.

> **Official UE5.4 Docs Reference:** Timers in Blueprints
> https://dev.epicgames.com/documentation/en-us/unreal-engine/timer-nodes-in-unreal-engine

---

## SECTION 8 — WIRING INPUTS IN BP_PlayerCharacter

**You are ADDING nodes to BP_PlayerCharacter, not replacing anything. Coordinate with your teammate before opening this file.**

### What You're Adding

1. Add `AC_MovementExtComponent` to the character's Components panel
2. Add `AC_UtilityComponent` to the character's Components panel
3. Wire `IA_Grapple` input → `TryGrapple()` call on your component
4. Wire `IA_Utility` input → `ThrowSatchel()` call on your component

### ⚠️ IMPORTANT — Avoid Breaking Your Teammate's Work

- **Do NOT delete any existing nodes**
- **Do NOT change the BeginPlay sequence your teammate already built**
- **Add your input bindings at the END of the existing input chain**
- **Best practice:** Make an appointment with your teammate and do this step together at one computer

### Step-by-Step

#### Step 1 — Open BP_PlayerCharacter

Located at: `Content/ProjectAnomaly/Player/Character/BP_PlayerCharacter`

#### Step 2 — Add Your Components

In the **Components** panel:
1. Click **+ Add** → search `AC_MovementExtComponent` → add it → rename to `MovementExtComponent`
2. Click **+ Add** → search `AC_UtilityComponent` → add it → rename to `UtilityComponent`

#### Step 3 — Wire IA_Grapple Input

1. In the **Event Graph**, right-click → `Enhanced Action Events` → search `IA_Grapple`
2. Select it — you get an `IA_Grapple` input event node with `Started`, `Ongoing`, `Triggered`, `Canceled`, `Completed` exec pins
3. From the **Started** exec pin:
   - Drag from it → right-click → search `Get MovementExtComponent` (your variable/component)
   - From the component → drag → `TryGrapple`
   - Connect Started exec → TryGrapple

#### Step 4 — Wire IA_Utility Input

1. Right-click → `Enhanced Action Events` → search `IA_Utility`
2. From the **Started** exec pin:
   - `Get UtilityComponent` → `ThrowSatchel`
   - Connect Started exec → ThrowSatchel

#### Step 5 — Compile and Save

Compile → green checkmark → Save.

> **Official UE5.4 Docs Reference:** Actor Components in Blueprints
> https://dev.epicgames.com/documentation/en-us/unreal-engine/components-in-unreal-engine

---

## SECTION 9 — DEBUG & TEST CHECKLIST

**Run through every test below in order. Fix each one before moving on.**

### 9.1 — Grapple Tests

| # | What to Test | How to Test | Expected Result | Common Failure |
|---|---|---|---|---|
| G-01 | Grapple compiles | Open AC_MovementExtComponent → Compile | Green checkmark, no errors | Missing BP_GrappleAnchor reference — create it first |
| G-02 | IA_Grapple input fires | Add `Print String "GRAPPLE"` at the IA_Grapple Started pin → Play → press E | Prints "GRAPPLE" | IMC_Default not registering — check BP_PlayerController has Add Mapping Context in BeginPlay |
| G-03 | Line trace fires | Temporarily add `Draw Debug Line` at the same point in TryGrapple. Play → press E → look around | See a debug line shooting from camera | No line = component not attached to character |
| G-04 | Grapple hits anchor and launches | Aim at a placed BP_GrappleAnchor → press E | Player shoots toward the anchor | Line trace not hitting: check anchor's collision is set to Visibility channel |
| G-05 | Grapple misses non-anchor surfaces | Aim at a wall → press E | Nothing happens, player stays still | Player launches into wall = Cast to BP_GrappleAnchor not filtering correctly |
| G-06 | bXYOverride + bZOverride both true | Check the Launch Character node in TryGrapple | Player moves in all 3 axes toward anchor | Player only moves horizontally = Z override is false |

> **Enable Debug Line Traces:**
> Add a `Draw Debug Line` node anywhere in TryGrapple:
> - Start: Camera Location
> - End: Camera Location + (Forward × GrappleRange)
> - Color: Red
> - Duration: 2.0 seconds
> This shows you exactly where the trace goes each time you press E.

### 9.2 — Satchel Tests

| # | What to Test | How to Test | Expected Result | Common Failure |
|---|---|---|---|---|
| S-01 | BP_SatchelCharge compiles | Open BP_SatchelCharge → Compile | Green checkmark | FS_DamageInfo struct doesn't exist or field names mismatched |
| S-02 | AC_UtilityComponent compiles | Open AC_UtilityComponent → Compile | Green checkmark | BP_SatchelCharge class not found — make sure it compiled first |
| S-03 | Satchel spawns on Q press | Add Print String "THROWN" at ThrowSatchel start → Play → press Q | Prints "THROWN" | IMC_Default not mapping Q to IA_Utility |
| S-04 | Satchel is a physics object | Press Q → watch the thrown satchel | It follows a projectile arc and bounces | Simulate Physics not enabled on Sphere Collision |
| S-05 | Fuse detonates after 2s | Add Print String "DETONATED" in Detonate event → Play → throw → watch | Prints "DETONATED" ~2 seconds after throw | Function Name in Set Timer doesn't match "Detonate" exactly |
| S-06 | Blast jump works | Stand near the satchel when it detonates | Player is launched upward | bImpulseVelChange=false or bIgnoreOwningActor=true on RadialForceComponent |
| S-07 | Satchel damages enemies | Place a BP_Enemy_Brute (if built) or any BPI_Damageable actor near detonation | Enemy health decreases | BPI_Damageable interface not implemented on enemy, or distance check wrong |
| S-08 | Cooldown works | Throw → immediately try to throw again | Second throw does nothing | bCanThrow not set to false after throw |
| S-09 | Cooldown resets after 45s | Wait 45+ seconds after throw → press Q | New satchel spawns | StartCooldown event name not matching timer function name |

### 9.3 — Integration Tests (After Both Systems Are Wired)

| # | Test | Expected |
|---|---|---|
| I-01 | Grapple to high anchor → satchel mid-air | Both actions work independently without interfering |
| I-02 | Throw satchel → grapple away before detonation | Both still work — blast jump catches you mid-grapple |
| I-03 | Full rage loop (if AC_RageComponent exists) | Grapple works outside rage, dash stub does nothing |
| I-04 | Die and respawn | Both cooldowns reset, grapple and satchel work again from respawn |

---

## SECTION 10 — GITHUB HANDOFF

**Before pushing your branch, confirm your file list and make sure you haven't modified anything you shouldn't have.**

### Files You Authored (Safe to Push)

```
Content/ProjectAnomaly/Arena/BP_GrappleAnchor.uasset
Content/ProjectAnomaly/Player/Components/AC_MovementExtComponent.uasset
Content/ProjectAnomaly/Combat/BP_SatchelCharge.uasset
Content/ProjectAnomaly/Player/Components/AC_UtilityComponent.uasset
Content/ProjectAnomaly/Input/IA_Grapple.uasset           (if you created it)
Content/ProjectAnomaly/Input/IA_Utility.uasset           (if you created it)
Content/ProjectAnomaly/Data/Structs/FS_DamageInfo.uasset (if you created it)
Content/ProjectAnomaly/Interfaces/BPI_Damageable.uasset  (if you created it)
```

### Files You Modified (Coordinate on These)

```
Content/ProjectAnomaly/Player/Character/BP_PlayerCharacter.uasset
Content/ProjectAnomaly/Input/IMC_Default.uasset
```

For these files, **both you and your teammate edited them**. Unreal `.uasset` files are binary — git cannot auto-merge them. You need to:
1. **One person acts as integrator** — opens the file and manually adds both sets of changes
2. Or use **Unreal's built-in revision control** integration (Perforce / Plastic SCM) for safer binary merges

**Recommended merge workflow:**
1. Your teammate pushes their branch first
2. You pull their changes
3. You open `BP_PlayerCharacter` → add your component references and input events on top of their work
4. Push your branch
5. One person does the final PR merge on GitHub

### What to Tell Your Teammate About AC_MovementExtComponent

When you hand off `AC_MovementExtComponent`:
- Tell them `TryDash()` is an empty stub function with a `Direction (Vector)` input — they need to implement the body
- Tell them the `bDashAvailable` variable exists and is set to false — they should wire their rage binding to set it true on `OnEnragedStateBegin` and false on `OnEnragedStateEnd`
- Tell them `OnDashUsed` dispatcher exists — they should call it when dash fires

---

## SECTION 11 — GLOSSARY

**Quick definitions for UE5 terms used in this guide.**

| Term | What It Means |
|---|---|
| **Actor Component (AC_)** | A modular piece of logic attached to a Blueprint Actor. Can't exist alone — always lives on an Actor. |
| **Blueprint Actor (BP_)** | A full game object that can be placed in the world (e.g. BP_SatchelCharge) |
| **Line Trace** | UE's raycast — fires an invisible ray in a direction and returns what it hits |
| **Launch Character** | UE function that overrides the Character Movement Component's velocity to throw the player in a direction |
| **Radial Force Component** | A component that pushes nearby physics objects outward in a sphere — perfect for explosions |
| **bImpulseVelChange** | Setting on Radial Force that makes the push physics-mass-independent — heavy things and light things get thrown equally far |
| **Event Dispatcher** | A broadcast system in UE Blueprints. When you "Call" a dispatcher, every function bound to it fires. Like a radio broadcast — one sender, many listeners. |
| **Custom Event** | A named entry point in a Blueprint graph that can be called by name (e.g. by timers) or by other Blueprints |
| **Timer by Function Name** | Calls a Custom Event by its string name after a delay. Function Name must match exactly. |
| **Cast to** | A Blueprint node that checks if an object is a specific type and gives you access to that type's variables and functions |
| **Instance Editable** | Makes a variable editable per-instance in the level editor — so you can set GrappleSpeed to different values on different instances |
| **Sphere Collision** | A sphere-shaped collision volume component — used as the satchel's physics body and the grapple anchor's detection volume |
| **BPI_ (Blueprint Interface)** | A contract between Blueprints. If an actor implements BPI_Damageable, you can call ReceiveDamage on it without knowing what type of actor it is. |
| **FS_ (Struct)** | A named grouping of multiple variables. FS_DamageInfo bundles all damage info into one package passed around between systems. |

---

## APPENDIX A — DEPENDENCY ORDER FOR YOUR SYSTEMS

Build in this exact order to avoid compile errors:

```
Step 1:  FS_DamageInfo         (no dependencies)
Step 2:  BPI_Damageable        (no dependencies)
Step 3:  IA_Grapple            (no dependencies)
Step 4:  IA_Utility            (no dependencies)
Step 5:  IMC_Default           (needs IA_ assets to exist)
Step 6:  BP_GrappleAnchor      (no dependencies)
Step 7:  BP_SatchelCharge      (needs FS_DamageInfo, BPI_Damageable)
Step 8:  AC_MovementExtComponent (needs BP_GrappleAnchor)
Step 9:  AC_UtilityComponent   (needs BP_SatchelCharge)
Step 10: Wire BP_PlayerCharacter (needs all above + teammate's components)
```

---

## APPENDIX B — KEY VALUES AT A GLANCE

All these values are Instance Editable — you can change them without reopening Blueprint logic.

| Variable | Default | Where | Change Via |
|---|---|---|---|
| GrappleSpeed | 2500.0 | AC_MovementExtComponent | Select component in player → Details panel |
| GrappleRange | 3000.0 | AC_MovementExtComponent | Select component in player → Details panel |
| ThrowForce | 2000.0 | AC_UtilityComponent | Select component in player → Details panel |
| CooldownDuration | 45.0 | AC_UtilityComponent | Select component in player → Details panel |
| BaseDamage (Satchel) | 100.0 | BP_SatchelCharge | Select instance in level → Details panel |
| BlastRadius | 600.0 | BP_SatchelCharge | Select instance in level → Details panel |
| FuseDuration | 2.0 | BP_SatchelCharge | Select instance in level → Details panel |
| BlastForce Strength | 200000.0 | BP_SatchelCharge RadialForce | Open BP_SatchelCharge → select BlastForce component |

---

## APPENDIX C — OFFICIAL UE5.4 DOCUMENTATION LINKS

| Topic | URL |
|---|---|
| Enhanced Input System | https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine |
| Blueprint Interfaces | https://dev.epicgames.com/documentation/en-us/unreal-engine/implementing-blueprint-interfaces-in-unreal-engine |
| Line Traces (Raycasting) | https://dev.epicgames.com/documentation/en-us/unreal-engine/traces-in-unreal-engine---overview |
| Actor Components | https://dev.epicgames.com/documentation/en-us/unreal-engine/components-in-unreal-engine |
| Timers in Blueprints | https://dev.epicgames.com/documentation/en-us/unreal-engine/timer-nodes-in-unreal-engine |
| Radial Force Component | https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Components/URadialForceComponent |
| Structs in Blueprints | https://dev.epicgames.com/documentation/en-us/unreal-engine/blueprint-struct-variables-in-unreal-engine |
| Spawn Actor From Class | https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/Kismet/UGameplayStatics/SpawnObject |
| Launch Character | https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/GameFramework/ACharacter/LaunchCharacter |
| Data Tables | https://dev.epicgames.com/documentation/en-us/unreal-engine/data-driven-gameplay-elements-in-unreal-engine |

---

*Guide authored for: Project Anomaly — Vansh's Branch*
*Based on: GDD v1.1 · TDD v1.1 · Blueprint Guides P1/P2/P3 · Architecture Document · Dependency Map*
*Engine: Unreal Engine 5.4.4*
