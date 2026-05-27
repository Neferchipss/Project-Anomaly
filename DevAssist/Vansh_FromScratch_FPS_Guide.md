# PROJECT ANOMALY — GRAPPLE + SATCHEL FROM SCRATCH
## Starting From Default UE5 FPS Template · UE 5.4.4 · Branch: Vansh

> **ADHD-FRIENDLY FORMAT:** Every section opens with one sentence saying exactly what you are doing and why.
> **⚠️ STOP** = do not continue until this is resolved.
> **✅ CHECK** = verify this before moving on.
> **🔧 DEBUG** = exact test to run after each build step.

---

## QUICK NAVIGATION

| I want to... | Jump to |
|---|---|
| See what the FPS template already gives me | [Section 1 — Starting State](#section-1--starting-state-fps-template) |
| Understand exactly what I'm building | [Section 2 — What You're Building](#section-2--what-youre-building) |
| See the build order at a glance | [Section 3 — Build Order](#section-3--build-order) |
| Create the folder structure | [Section 4 — Folder Setup](#section-4--folder-setup) |
| Build FS_DamageInfo struct | [Section 5 — FS_DamageInfo](#section-5--fs_damageinfo-struct) |
| Build BPI_Damageable interface | [Section 6 — BPI_Damageable](#section-6--bpi_damageable-interface) |
| Set up input actions | [Section 7 — Input Actions](#section-7--input-actions--key-bindings) |
| Build the grapple target marker | [Section 8 — BP_GrappleAnchor](#section-8--bp_grappleanchor) |
| Build the grapple logic component | [Section 9 — AC_MovementExtComponent](#section-9--ac_movementextcomponent) |
| Build the satchel projectile | [Section 10 — BP_SatchelCharge](#section-10--bp_satchelcharge) |
| Build the satchel manager component | [Section 11 — AC_UtilityComponent](#section-11--ac_utilitycomponent) |
| Wire everything into the player | [Section 12 — Wiring the Character](#section-12--wiring-bp_firstpersoncharacter) |
| Test that everything works | [Section 13 — Full Test Checklist](#section-13--full-test-checklist) |
| Look up all the tunable values | [Section 14 — Key Values](#section-14--key-values-at-a-glance) |
| Look up a term | [Section 15 — Glossary](#section-15--glossary) |

---

## SECTION 1 — STARTING STATE (FPS TEMPLATE)

**Before you build anything, know exactly what the FPS template already provides so you don't duplicate it.**

When you create a new UE5 project with the First Person template, you get:

| Already Exists | Location | Relevant to You? |
|---|---|---|
| `BP_FirstPersonCharacter` | `Content/FirstPerson/Blueprints/` | YES — you will add your components here |
| `IMC_Default` (input mapping context) | `Content/FirstPerson/Input/` | YES — you will ADD new bindings to this |
| `IA_Move`, `IA_Look`, `IA_Jump`, `IA_Fire` | `Content/FirstPerson/Input/Actions/` | NO — don't touch these |
| A First Person Camera already attached | Inside `BP_FirstPersonCharacter` | YES — your grapple trace uses it |
| Arms skeletal mesh with socket `GripPoint` | Inside `BP_FirstPersonCharacter` | YES — satchel spawns from here |
| Enhanced Input subsystem already wired | Inside `BP_FirstPersonCharacter` BeginPlay | YES — your new IAs hook into the same IMC |

### What the FPS template does NOT have (you build these):

- `FS_DamageInfo` struct
- `BPI_Damageable` interface
- `IA_Grapple` and `IA_Utility` input actions
- `BP_GrappleAnchor` actor
- `AC_MovementExtComponent` (TryGrapple logic)
- `BP_SatchelCharge` actor
- `AC_UtilityComponent` (ThrowSatchel + cooldown)

> **✅ CHECK — FPS Template Camera Setup:**
> Open `BP_FirstPersonCharacter` → Components panel → confirm you see a `FirstPersonCameraComponent`. The grapple line trace fires from this. If you see it, you're good.

---

## SECTION 2 — WHAT YOU'RE BUILDING

**Two complete systems that give the player a movement ability (grapple) and a tactical explosive (satchel).**

### Grapple Hook

Press **E** → fires an invisible ray from the camera → if it hits a `BP_GrappleAnchor` actor placed in the level → player launches through the air toward it at 2500 units/sec.

What it uses in UE terms:
- **Line Trace By Channel** (the invisible ray)
- **Cast to BP_GrappleAnchor** (confirms the ray hit a valid target)
- **Launch Character** (physically moves the player)
- All owned by **AC_MovementExtComponent**

### Satchel Charge

Press **Q** → spawns a small physics ball (`BP_SatchelCharge`) that flies forward → bounces off surfaces → after 2 seconds detonates → pushes and damages nearby actors in a 600-unit radius → 45-second cooldown before you can throw again.

What it uses in UE terms:
- **Simulate Physics** (the ball bounces naturally)
- **Set Timer** (the 2-second fuse)
- **RadialForceComponent → Fire Impulse** (the shockwave that blast-jumps you)
- **BPI_Damageable → ReceiveDamage** (the interface call that deals damage)
- Owned by **AC_UtilityComponent** (spawns it) and **BP_SatchelCharge** (detonates itself)

### Your 5 Deliverables

| Asset | Type | Folder |
|---|---|---|
| `BP_GrappleAnchor` | Blueprint Actor | `Content/ProjectAnomaly/Arena/` |
| `AC_MovementExtComponent` | Actor Component | `Content/ProjectAnomaly/Player/Components/` |
| `BP_SatchelCharge` | Blueprint Actor | `Content/ProjectAnomaly/Combat/` |
| `AC_UtilityComponent` | Actor Component | `Content/ProjectAnomaly/Player/Components/` |
| Input wiring | Inside `BP_FirstPersonCharacter` | `Content/FirstPerson/Blueprints/` |

---

## SECTION 3 — BUILD ORDER

**Build these in EXACTLY this order. Earlier items have no dependencies; later items will fail to compile without them.**

```
1.  Folder Structure         (no dependencies — just folders)
2.  FS_DamageInfo            (no dependencies — pure data)
3.  BPI_Damageable           (no dependencies — pure interface)
4.  IA_Grapple               (no dependencies — just an Input Action asset)
5.  IA_Utility               (no dependencies — just an Input Action asset)
6.  IMC_Default bindings     (needs IA_Grapple and IA_Utility to exist first)
7.  BP_GrappleAnchor         (no dependencies — pure target marker)
8.  BP_SatchelCharge         (needs FS_DamageInfo and BPI_Damageable)
9.  AC_MovementExtComponent  (needs BP_GrappleAnchor)
10. AC_UtilityComponent      (needs BP_SatchelCharge)
11. Wire BP_FirstPersonChar  (needs everything above + IMC already wired in BeginPlay)
```

> **⚠️ STOP — Do not skip steps.** If you try to create `AC_MovementExtComponent` before `BP_GrappleAnchor` exists, the Cast node will have a missing reference and the blueprint will show a compile error immediately.

---

## SECTION 4 — FOLDER SETUP

**Create the folder structure first so every asset goes in the right place from the start.**

In the **Content Browser**:

1. Right-click in the **Content** root → **New Folder** → name it `ProjectAnomaly`
2. Double-click into `ProjectAnomaly/`
3. Create these sub-folders one by one (right-click → New Folder for each):
   - `Arena`
   - `Combat`
   - `Data`
   - `Interfaces`
   - `Input`
   - `Player`
4. Double-click into `Player/` → create one sub-folder:
   - `Components`
5. Double-click into `Data/` → create one sub-folder:
   - `Structs`

Your final structure inside Content/ProjectAnomaly/:
```
Content/ProjectAnomaly/
├── Arena/
├── Combat/
├── Data/
│   └── Structs/
├── Input/
├── Interfaces/
└── Player/
    └── Components/
```

> **✅ CHECK:** Click on `Content/ProjectAnomaly/Player/Components/` — if it opens to an empty folder with no errors, the folder structure is correct.

---

## SECTION 5 — FS_DamageInfo STRUCT

**This struct is the universal damage package. Every system that deals damage sends one of these. Build it before anything else or nothing will compile.**

### What it is

A Blueprint Structure is a named group of variables bundled together — like a form with 6 fields. Instead of passing 6 separate values between systems, you pass one `FS_DamageInfo`.

### Fields Required

| Field Name | Type | Notes |
|---|---|---|
| `BaseDamage` | Float | How much damage to deal |
| `bIsWeakPointHit` | Bool | Was this a weak point hit? |
| `bIsArmourDamage` | Bool | Does this hit armour? |
| `bIsNextShotBoosted` | Bool | Is this a boosted shot (rage)? |
| `Instigator` | Object Reference → Actor | Who caused this damage? |
| `HitLocation` | Vector | Where did the hit land? |

> **⚠️ STOP — Field names are case-sensitive.** `BaseDamage` works. `baseDamage` or `Base_Damage` will cause silent failures in every system that uses this struct.

### Step-by-Step

1. In Content Browser, navigate to `Content/ProjectAnomaly/Data/Structs/`
2. Right-click in the empty folder → **Blueprint** → **Structure**
3. Name it exactly: `FS_DamageInfo`
4. Double-click to open it
5. You will see a single default variable. Rename it to `BaseDamage` and set its type to **Float**
6. Click the **+** button (top right of the Members panel) 5 more times to add 5 more fields
7. Set each new field name and type according to the table above:
   - Field 2: `bIsWeakPointHit` → type: **Bool**
   - Field 3: `bIsArmourDamage` → type: **Bool**
   - Field 4: `bIsNextShotBoosted` → type: **Bool**
   - Field 5: `Instigator` → type: **Object Reference** → in the dropdown search for **Actor** → select it
   - Field 6: `HitLocation` → type: **Vector**
8. Click **Save** (top-left toolbar)

> **✅ CHECK:** After saving, your struct should show exactly 6 members in the panel. No compile step needed for structs — Save is enough.

---

## SECTION 6 — BPI_DAMAGEABLE INTERFACE

**This interface is what allows the satchel to deal damage to any actor without needing to know what type of actor it is. Build it before BP_SatchelCharge.**

### What it is

A Blueprint Interface defines function signatures — contracts. Any actor that implements `BPI_Damageable` is saying "I can receive damage." The satchel calls `ReceiveDamage` on nearby actors through this interface without caring if they're enemies, destructible props, or the player.

### Functions Required

| Function Name | Inputs | Return |
|---|---|---|
| `ReceiveDamage` | `DamageInfo` (FS_DamageInfo) | None |
| `IsAlive` | None | Bool |
| `GetCurrentHealth` | None | Float |

### Step-by-Step

1. Navigate to `Content/ProjectAnomaly/Interfaces/`
2. Right-click → **Blueprint** → **Blueprint Interface**
3. Name it exactly: `BPI_Damageable`
4. Double-click to open it
5. You will see a default function called `NewFunction_0`. Rename it to `ReceiveDamage`
6. With `ReceiveDamage` selected in the left panel, look at the right panel for **Inputs**
7. Click **+** under Inputs → name it `DamageInfo` → set its type to `FS_DamageInfo` (search for it)
8. In the left panel, click **+** to add a second function → name it `IsAlive`
9. With `IsAlive` selected → look at **Outputs** → click **+** → name it `ReturnValue` → type: **Bool**
10. Click **+** for a third function → name it `GetCurrentHealth`
11. With `GetCurrentHealth` selected → **Outputs → +** → name `ReturnValue` → type: **Float**
12. Click **Save**

> **⚠️ STOP:** Interface functions have no graph (no nodes inside them). They are just signatures. This is correct. The actual implementation lives in each actor that uses the interface. Do not try to add nodes.

---

## SECTION 7 — INPUT ACTIONS + KEY BINDINGS

**You need two new Input Action assets, then bind them to keys inside the FPS template's existing IMC_Default.**

### Step 1 — Create IA_Grapple

1. Navigate to `Content/ProjectAnomaly/Input/`
2. Right-click → **Input** → **Input Action**
3. Name it exactly: `IA_Grapple`
4. Double-click to open it
5. In Details panel → **Value Type** → set to **Digital (bool)**
6. Everything else stays default
7. Click **Save**

### Step 2 — Create IA_Utility

1. Still in `Content/ProjectAnomaly/Input/`
2. Right-click → **Input** → **Input Action**
3. Name it exactly: `IA_Utility`
4. Double-click to open it
5. **Value Type** → **Digital (bool)**
6. Click **Save**

### Step 3 — Add Bindings to IMC_Default

The FPS template already has `IMC_Default` at `Content/FirstPerson/Input/IMC_Default`. You are **adding** to it, not replacing it.

1. Navigate to `Content/FirstPerson/Input/`
2. Double-click `IMC_Default` to open it
3. You will see existing mappings for Fire, Jump, Look, Move — **do not touch these**
4. Click **+** (Add Mapping) at the top → a new empty row appears
5. Click the dropdown in the new row → search `IA_Grapple` → select it
6. Click the **+** next to IA_Grapple → a key slot appears
7. Click the key slot → press **E** on your keyboard → it should register as **E**
8. Click **+** again to add another new mapping
9. Set it to `IA_Utility`
10. Add a key slot → press **Q**
11. Click **Save**

> **✅ CHECK:** Your IMC_Default should now show at minimum 6 mappings: Move, Look, Jump, Fire, IA_Grapple (E), IA_Utility (Q).

> **🔧 DEBUG — Confirm Input Fires:**
> Before building any components, open `BP_FirstPersonCharacter` Event Graph → right-click → search `Enhanced Action Events` → search `IA_Grapple` → add the node → from `Started` pin → right-click → `Print String` → type "GRAPPLE PRESSED" → Compile → Play → press E → should print. Delete this test node when done.

---

## SECTION 8 — BP_GRAPPLEANCHOR

**The simplest asset you will build. It is a pure position marker with no logic — just a sphere the line trace can hit.**

### What It Does

Nothing on its own. It just exists in the level at positions you choose. The grapple system fires a line trace and casts the hit result — if it's a `BP_GrappleAnchor`, the grapple succeeds. If it's a wall or floor, nothing happens.

### Step-by-Step

#### Create the Blueprint

1. Navigate to `Content/ProjectAnomaly/Arena/`
2. Right-click → **Blueprint Class**
3. In the class picker, type `Actor` in the search box → select **Actor** → click **Select**
4. Name it exactly: `BP_GrappleAnchor`
5. Double-click to open it

#### Add a Sphere Collision Component

1. In the **Components** panel (top-left of the Blueprint editor), click **+ Add**
2. Type `Sphere Collision` in the search → select it
3. With the new sphere selected, find the **Details** panel (right side):
   - **Sphere Radius** → type `50.0`
   - Scroll down to **Collision** section
   - **Collision Presets** → click the dropdown → select `OverlapAllDynamic`
4. Rename the component: click its name in Components panel → type `GrappleSphere`

> **Why OverlapAllDynamic?** Line traces on the Visibility channel detect meshes and collision shapes. OverlapAllDynamic ensures the sphere is visible to the trace even though it doesn't block movement.

#### Add a Visible Mesh (So You Can See It in the Editor)

1. Click **+ Add** again → type `Static Mesh` → select it
2. Rename it `AnchorMesh`
3. In Details panel → **Static Mesh** → click the dropdown → search `SM_Sphere` → if no results, enable **Show Engine Content** (bottom-left of the picker) → select `SM_Sphere`
4. With AnchorMesh still selected → **Transform → Scale** → set X, Y, Z all to `0.3`

> **There are ZERO Blueprint nodes in this actor.** Do not add any. Click **Compile** → green checkmark → click **Save**.

#### Place Anchors in the Level

1. Open your test level (the one the FPS template starts you in, or any level)
2. In the **Content Browser**, find `BP_GrappleAnchor`
3. Drag it into the viewport
4. Place at least **3 anchors** at different heights:
   - One at Z ≈ 300 (low)
   - One at Z ≈ 600 (mid)
   - One at Z ≈ 1000+ (high)
5. Make sure none are inside walls

> **🔧 DEBUG:** In the viewport, look for the small white sphere at each anchor position. If you can't see them, they may be inside geometry. Move them into open air.

---

## SECTION 9 — AC_MOVEMENTEXTCOMPONENT

**This component owns the TryGrapple function. It lives on the player character and fires a line trace when called.**

### What It Does

- `TryGrapple()` — fires a line trace from the camera. If it hits a `BP_GrappleAnchor`, stores it and calls `Launch Character` toward it.
- `TryDash()` — empty stub function left for your teammate to implement later.

### Variables You Will Create

| Variable Name | Type | Default | Instance Editable |
|---|---|---|---|
| `CurrentAnchor` | Object Reference → `BP_GrappleAnchor` | None | No |
| `GrappleSpeed` | Float | `2500.0` | Yes |
| `GrappleRange` | Float | `3000.0` | Yes |
| `bIsGrappling` | Bool | false | No |
| `bDashAvailable` | Bool | false | No |

### Event Dispatchers You Will Create

| Dispatcher Name | Payload | Purpose |
|---|---|---|
| `OnGrappleAttached` | None | Animation system listens to this |
| `OnDashUsed` | None | Animation system listens to this |

---

### Step-by-Step Build

#### Step 1 — Create the Component

1. Navigate to `Content/ProjectAnomaly/Player/Components/`
2. Right-click → **Blueprint Class**
3. In the class picker, type `ActorComponent` → select **Actor Component** → click **Select**
4. Name it exactly: `AC_MovementExtComponent`
5. Double-click to open it

#### Step 2 — Add Variables

In the **My Blueprint** panel (left side of Blueprint editor), find the **Variables** section.

Click **+** 5 times and configure each one:

**Variable 1:**
- Name: `CurrentAnchor`
- Click the type dropdown (currently says "Boolean") → search `BP_GrappleAnchor` → select **BP_GrappleAnchor** → then click the small icon next to it (the one with an arrow) and pick **Object Reference**
- Instance Editable: leave unchecked

**Variable 2:**
- Name: `GrappleSpeed`
- Type: **Float**
- In Details panel → Default Value: `2500.0`
- Instance Editable: **check the box** (eye icon or checkbox in Details)

**Variable 3:**
- Name: `GrappleRange`
- Type: **Float**
- Default Value: `3000.0`
- Instance Editable: **checked**

**Variable 4:**
- Name: `bIsGrappling`
- Type: **Boolean**
- Default Value: false (unchecked)
- Instance Editable: leave unchecked

**Variable 5:**
- Name: `bDashAvailable`
- Type: **Boolean**
- Default Value: false
- Instance Editable: leave unchecked

> **How to set Instance Editable:** With the variable selected → look for **Instance Editable** in the Details panel → check the box. This lets you change the value per-instance in the level editor without reopening the Blueprint.

#### Step 3 — Add Event Dispatchers

In My Blueprint panel → find the **Event Dispatchers** section → click **+** twice:

1. Name the first dispatcher: `OnGrappleAttached` (no inputs needed)
2. Name the second: `OnDashUsed` (no inputs needed)

Click **Compile** after adding them — should be clean so far.

#### Step 4 — Build the TryGrapple Function

In My Blueprint → **Functions** section → click **+** → name the new function `TryGrapple`.

The function graph opens. You are now building inside this function. Here is every node in order:

---

**Node 1: Get Owner**
- Right-click in empty graph space → search `Get Owner` → select it
- This returns the actor that this component is attached to (the player character)

**Node 2: Cast to Character**
- Drag the white **Return Value** pin from Get Owner → release → search `Cast to Character` → select it
- Connect Get Owner's execution (white) pin → Cast to Character's execution pin
- Connect Get Owner's Return Value → Cast to Character's Object input

**Node 3: Get Controller**
- From `Cast to Character`'s `As Character` output pin → drag → search `Get Controller` → select it

**Node 4: Cast to PlayerController**
- Drag Controller's Return Value → search `Cast to PlayerController` → select it
- Connect the execution chain: Cast to Character success → Cast to PlayerController

**Node 5: Get Player View Point**
- From `As Player Controller` output pin → drag → search `Get Player View Point` → select it
- This gives you two outputs: `Location` (where the camera is) and `Rotation` (which way the camera faces)

**Node 6: Get Forward Vector**
- From the `Rotation` output of Get Player View Point → drag → search `Get Forward Vector` → select it
- This converts the camera's rotation into a normalized direction vector pointing forward

**Node 7: Compute the End point of the line trace**

You need: `CameraLocation + (ForwardVector × GrappleRange)`

First, multiply:
- Right-click in graph → search `float * vector` → look for **Float * Vector** (multiply node) → add it
- Connect `GrappleRange` variable (drag it from My Blueprint Variables panel) to the **float** input pin
- Connect Get Forward Vector's output to the **vector** input pin

Then, add:
- Right-click → search `vector + vector` → select the **Add (Vector)** node
- Connect the `Location` output from Get Player View Point to one input
- Connect the multiply result to the other input
- This gives you the end point of your trace

**Node 8: Line Trace By Channel**
- Right-click → search `Line Trace By Channel` → select it
- **Start** pin → connect `Location` from Get Player View Point
- **End** pin → connect the vector addition result from Node 7
- **Trace Channel** → click dropdown → select `Visibility`
- Leave `bTraceComplex` as false, ignore other options
- Connect execution: Cast to PlayerController success → Line Trace

**Node 9: Branch on bBlockingHit**
- From Line Trace → drag the `Out Hit` pin → search `Break Hit Result` → add it
- From Line Trace's execution → drag → search `Branch` → add it
- From `Break Hit Result` → connect `Blocking Hit` (bool) to the **Condition** pin of Branch

**Node 10: Handle FALSE branch**
- From Branch's **False** exec pin → right-click → `Return Node`
- This is the silent exit: player aimed at something that wasn't a grapple anchor

**Node 11: Cast to BP_GrappleAnchor**
- From Branch's **True** exec pin → drag → `Cast to BP_GrappleAnchor` → add it
- From `Break Hit Result` → connect the **Hit Actor** output to the Cast's Object input

**Node 12: Handle Cast Failure**
- From Cast to BP_GrappleAnchor's **Cast Failed** exec pin → `Return Node`
- Silent exit: line trace hit something, but it was a wall, not an anchor

**Node 13: SET CurrentAnchor**
- From Cast to BP_GrappleAnchor's **success** exec pin → drag a **Set** variable node for `CurrentAnchor`
  - In My Blueprint, right-click `CurrentAnchor` → drag into graph → select **Set CurrentAnchor**
- Connect the execution from Cast success → SET CurrentAnchor
- Connect `As BP_GrappleAnchor` output from the Cast node → into the SET node's input value

**Node 14: Get Anchor Location**
- From `As BP_GrappleAnchor` output → drag → search `Get Actor Location` → add it
- This gives you the 3D position of the anchor in the world

**Node 15: Compute Launch Direction**
- Need: `Normalize(AnchorLocation - PlayerLocation)`

Get player location:
- Right-click → `Get Owner` → add another one (or reuse existing via a reroute node)
- From Get Owner → `Get Actor Location` → gives **PlayerLocation**

Subtract:
- Right-click → search `vector - vector` → **Subtract (Vector)** node
- First input: Anchor location (from Node 14)
- Second input: Player location (from Get Actor Location above)

Normalize:
- Right-click → search `Normalize Vector` → add it
- Connect the subtraction result to Normalize Vector's input
- Output is a unit vector pointing from player toward anchor — this is the launch direction

**Node 16: Compute Launch Velocity**
- Right-click → search `vector * float` → **Vector × Float** multiply node
- Connect Normalized direction to vector input
- Connect `GrappleSpeed` variable (drag from panel) to float input
- Output is the full velocity vector with correct magnitude

**Node 17: Launch Character**
- Get Owner + Cast to Character again (or reuse the variable if you stored it)
  - Right-click → `Get Owner` → `Cast to Character`
- From `As Character` → drag → search `Launch Character` → add it
- Connect execution from SET CurrentAnchor → Launch Character
- **Launch Velocity** pin → connect Node 16's output
- **bXYOverride** → **check it** (set to true) — overrides horizontal velocity
- **bZOverride** → **check it** (set to true) — overrides vertical velocity

> **⚠️ STOP — Both override booleans MUST be true.** If either is false, existing player movement velocity will interfere and the grapple will feel broken or go sideways.

**Node 18: SET bIsGrappling = true**
- After Launch Character execution → drag `bIsGrappling` from panel → **Set** node → check the bool (set to true)
- Connect execution: Launch Character → SET bIsGrappling

**Node 19: Call OnGrappleAttached Dispatcher**
- Drag `OnGrappleAttached` from My Blueprint Event Dispatchers → release in graph → select **Call**
- Connect execution: SET bIsGrappling → Call OnGrappleAttached

**Node 20: Return Node**
- After OnGrappleAttached → connect to a **Return Node** to cleanly exit

> **✅ CHECK — Compile now.** Click **Compile**. If you get an error about `BP_GrappleAnchor` missing, make sure you compiled and saved that actor first, then re-open this component and compile again.

---

#### Step 5 — Build the TryDash Stub

This is a placeholder for your teammate.

1. In My Blueprint → **Functions** → **+** → name it `TryDash`
2. In the function Details panel (when the function is selected) → **Inputs** → **+** → name: `Direction`, type: **Vector**
3. Inside the function graph: leave it completely empty (just the entry node)
4. Compile → this is valid — an empty function compiles fine

> **Note for teammate:** This is where you implement dash. Use `bDashAvailable` variable as the gate (Branch: bDashAvailable → Launch Character using the Direction input). Call `OnDashUsed` dispatcher when it fires.

#### Step 6 — Final Compile and Save

Click **Compile** → wait for green checkmark → click **Save**.

> **🔧 DEBUG — TryGrapple Quick Test:**
> Skip to Section 12, add the component to the character, wire IA_Grapple to TryGrapple, then:
> - Place 3 `BP_GrappleAnchor` actors in the level
> - Hit Play → aim at an anchor → press E
> - Expected: player launches toward anchor
> - If nothing happens: add a `Print String "GRAPPLE CALLED"` at the very start of TryGrapple to verify the function is being called at all
> - If player launches but wrong direction: check bXYOverride and bZOverride are both checked on Launch Character

---

## SECTION 10 — BP_SATCHELCHARGE

**This is the explosive physics ball. It handles its own flight, fuse timer, detonation, and self-destruction. Build this before AC_UtilityComponent.**

### What Happens at Runtime

1. `AC_UtilityComponent` spawns it at the player's hand position
2. An impulse pushes it forward — it flies and bounces (physics actor)
3. A 2-second timer starts at spawn (the fuse)
4. Timer fires → `Detonate` event runs:
   - `RadialForceComponent` fires impulse (blast-jumps the player and pushes others)
   - All nearby damageable actors receive `ReceiveDamage` via `BPI_Damageable`
   - Optional VFX spawns (placeholder for now)
   - Actor destroys itself

### Components Needed

| Component | Settings |
|---|---|
| `SatchelCollision` (Sphere Collision) | Radius: 15, Collision: PhysicsActor, Simulate Physics: TRUE, Enable Gravity: TRUE |
| `SatchelMesh` (Static Mesh) | Any sphere mesh, Scale: 0.15 on all axes |
| `BlastForce` (Radial Force) | Radius: 600, Strength: 200000, bImpulseVelChange: TRUE, bIgnoreOwningActor: FALSE |

### Variables Needed

| Variable | Type | Default |
|---|---|---|
| `BaseDamage` | Float | `100.0` |
| `BlastRadius` | Float | `600.0` |
| `FuseDuration` | Float | `2.0` |
| `InstigatorRef` | Object Reference → Actor | None |
| `FuseTimerHandle` | Timer Handle | (leave default) |

---

### Step-by-Step Build

#### Step 1 — Create the Blueprint

1. Navigate to `Content/ProjectAnomaly/Combat/`
2. Right-click → **Blueprint Class** → parent: **Actor** → name: `BP_SatchelCharge`
3. Double-click to open it

#### Step 2 — Add the Sphere Collision (Make It the Root)

1. In the **Components** panel → click **+ Add** → search `Sphere Collision` → select it
2. Name it `SatchelCollision`
3. In the Details panel:
   - **Sphere Radius**: `15.0`
   - **Collision Presets**: click the dropdown → `PhysicsActor`
   - Scroll down to **Physics** section:
     - **Simulate Physics**: check it ON
     - **Enable Gravity**: check it ON
4. Drag `SatchelCollision` to the top of the Components list (making it the root):
   - Right-click `SatchelCollision` → **Make Root Component**

> **⚠️ STOP — Simulate Physics MUST be on SatchelCollision.** If it's on the mesh instead, the satchel won't bounce correctly. Physics must be on the root collision component.

#### Step 3 — Add the Static Mesh

1. Click **+ Add** → `Static Mesh` → name it `SatchelMesh`
2. In Details → **Static Mesh** → pick `SM_Sphere` (enable Show Engine Content if needed)
3. **Transform → Scale**: X=0.15, Y=0.15, Z=0.15
4. Make sure `SatchelMesh` is parented under `SatchelCollision` in the hierarchy (drag it under in Components panel)

#### Step 4 — Add the Radial Force Component

1. Click **+ Add** → search `Radial Force` → select **Radial Force Component**
2. Name it `BlastForce`
3. In Details panel:
   - **Radius**: `600.0`
   - **Strength**: `200000.0`
   - **Impulse Strength**: `200000.0`
   - **bImpulseVelChange**: **check it ON** ← CRITICAL
   - **bIgnoreOwningActor**: **leave it UNCHECKED** (false) ← so it also pushes the player

> **Why bImpulseVelChange must be ON:** This makes the force apply as a direct velocity change rather than as a force over time. Without it, the player character (which has mass) barely moves. With it ON, the blast jump works correctly.
>
> **Why bIgnoreOwningActor must be OFF (false):** The owning actor of BlastForce is BP_SatchelCharge, not the player. Leaving it false is fine — you want the player to be pushed. The "owning actor" in this context is the satchel itself (which gets destroyed anyway).

#### Step 5 — Add Variables

In **My Blueprint → Variables**, click **+** 5 times:

1. `BaseDamage` → Float → Default: `100.0`
2. `BlastRadius` → Float → Default: `600.0`
3. `FuseDuration` → Float → Default: `2.0`
4. `InstigatorRef` → Object Reference → Actor → Default: none
5. `FuseTimerHandle` → **Timer Handle** type (search for Timer Handle in the type dropdown)

#### Step 6 — Build BeginPlay (Start the Fuse)

In the **Event Graph**, find the existing **Event BeginPlay** node.

From BeginPlay exec pin:
1. Right-click → search `Set Timer by Function Name` → add it
2. Configure the node:
   - **Object**: drag `Self` (right-click → `Self` or `Get Self Reference`) into the Object pin
   - **Function Name**: type the string `Detonate` — must be EXACTLY this with capital D
   - **Time**: drag the `FuseDuration` variable from My Blueprint into the Time pin
   - **Looping**: leave as false (unchecked)
3. From the Return Value pin of Set Timer → right-click → **Promote to Variable** → this creates `FuseTimerHandle` automatically (or connect it to your existing FuseTimerHandle SET node)
4. Connect BeginPlay exec → Set Timer by Function Name

> **⚠️ STOP — Function Name string MUST match.** The timer calls a Custom Event by its string name. If you later name the event `detonate` (lowercase) or `Detonate_` (with underscore), the timer will fire and silently fail to call anything.

#### Step 7 — Build the Detonate Event

In the Event Graph, right-click in empty space → **Add Custom Event** → name it `Detonate` (capital D, exact match).

Build the sequence from Detonate's exec pin:

---

**Part A — Fire the Blast Wave:**

1. In My Blueprint, drag the `BlastForce` component into the graph
2. From `BlastForce` → drag → search `Fire Impulse` → add it
3. Connect: Detonate exec → Fire Impulse
4. Connect: BlastForce variable → Fire Impulse's Target pin

---

**Part B — Deal Damage to Nearby Actors:**

The correct approach is to get ALL actors that implement `BPI_Damageable`, then filter by distance. Do NOT use `Get Overlapping Actors` — the collision radius is only 15 units; you need to damage in a 600 unit radius.

1. After Fire Impulse → right-click → search `Get All Actors with Interface` → add it
   - **Interface** dropdown → search `BPI_Damageable` → select it
   - Connect Fire Impulse exec → Get All Actors with Interface

2. From `Out Actors` pin → drag → search `For Each Loop` → add it
   - Connect Get All Actors with Interface exec → For Each Loop

3. Inside the loop — check distance, then deal damage if within blast radius:

   Get satchel's position:
   - Right-click → `Get Self` → `Get Actor Location` → this is the satchel's current position

   Get distance from satchel to current loop actor:
   - Right-click → search `Get Distance To` → add it
   - **Self** input: connect the satchel's Get Actor Location output... 
     Actually use this approach instead: right-click → **Vector Length** after subtracting positions
     
   Simpler approach using Get Distance To:
   - Drag `Self` reference into graph
   - From Self → `Get Distance To`
   - **Other Actor** input: connect `Array Element` from the For Each Loop
   - Output: a float representing the distance

   Branch on distance:
   - Right-click → `Branch` → Condition: Is distance `<=` BlastRadius?
   - Right-click → `float <= float` → first input: Get Distance To result, second input: `BlastRadius` variable
   - Connect the <= result to Branch condition

4. From Branch **True** exec → call ReceiveDamage via the interface:
   - Right-click → search for `Message ReceiveDamage` (look for the BPI_Damageable version — it will show the interface icon) → add it
   - **Target**: connect `Array Element` from the For Each loop
   - **Damage Info** input: right-click → `Make FS_DamageInfo` → configure each field:
     - `BaseDamage` → connect `BaseDamage` variable
     - `bIsWeakPointHit` → leave false (unchecked)
     - `bIsArmourDamage` → leave false
     - `bIsNextShotBoosted` → leave false
     - `Instigator` → connect `InstigatorRef` variable
     - `HitLocation` → right-click → `Get Actor Location` of Self → connect it

5. From Branch **False** exec → connect to the **For Each Loop**'s loop body end (the exec pin that goes to the next iteration — leave it unconnected, UE handles this automatically by connecting to the loop's "nothing" path back)

> Actually in UE Blueprints, the For Each Loop has:
> - **Loop Body** exec out: runs for each element
> - **Completed** exec out: runs when all elements processed
> You connect the Loop Body to your Branch node. The False path of Branch just falls through — leave it unconnected, it automatically moves to the next iteration.

6. After the `For Each Loop`'s **Completed** exec pin → continue to Part C

---

**Part C — Spawn VFX (Placeholder):**

1. After the For Each Loop Completed pin → right-click → `Spawn System at Location`
   - **System Template**: leave empty for now (connect `NS_SatchelExplosion` when VFX are built)
   - **Location**: `Get Actor Location` of Self
   - Connect the execution
   
> **Note:** If `NS_SatchelExplosion` doesn't exist yet, skip this node entirely or the blueprint will show a warning. Just continue to the Destroy step.

---

**Part D — Destroy the Satchel:**

1. After VFX spawn (or directly after the loop if skipping VFX) → right-click → `Destroy Actor`
   - **Target**: `Self` (leave as default — it defaults to Self)
   - Connect execution

---

**✅ Full Detonate sequence recap:**
```
[Detonate]
    → Fire Impulse (BlastForce)
    → Get All Actors with Interface (BPI_Damageable)
    → For Each Loop
        → [Loop Body] Get Distance To → Branch (≤ BlastRadius?)
            → [TRUE] Make FS_DamageInfo → Message ReceiveDamage
    → [Completed] Spawn VFX (optional)
    → Destroy Actor (Self)
```

#### Step 8 — Compile and Save

Click **Compile** → green checkmark → click **Save**.

> **🔧 DEBUG — Test Fuse and Detonate:**
> 1. Add `Print String "SATCHEL SPAWNED"` at the start of BeginPlay
> 2. Add `Print String "DETONATING"` at the start of Detonate
> 3. Place one `BP_SatchelCharge` directly in the level (drag from Content Browser)
> 4. Hit Play — after 2 seconds you should see "DETONATING" print
> 5. If "DETONATING" never prints: the Function Name in Set Timer doesn't match. Open BeginPlay → check the string is exactly `Detonate`
> 6. Remove the test Print String nodes when done

---

## SECTION 11 — AC_UTILITYCOMPONENT

**This component manages throwing the satchel, tracking the cooldown, and notifying the HUD when the player can throw again.**

### What It Does

- `ThrowSatchel()` — checks if `bCanThrow` is true → spawns `BP_SatchelCharge` at the player's hand → gives it a forward impulse → starts the 45-second cooldown timer → sets `bCanThrow = false`
- `StartCooldown` (Custom Event) — fires when 45 seconds have passed → sets `bCanThrow = true` again
- Two Event Dispatchers notify the HUD when throw starts and when cooldown ends

### Variables Needed

| Variable | Type | Default | Instance Editable |
|---|---|---|---|
| `bCanThrow` | Bool | true | No |
| `ThrowForce` | Float | `2000.0` | Yes |
| `CooldownDuration` | Float | `45.0` | Yes |
| `CooldownTimerHandle` | Timer Handle | default | No |

### Event Dispatchers Needed

| Dispatcher | Purpose |
|---|---|
| `OnSatchelThrown` | HUD binds to this to start showing cooldown |
| `OnCooldownComplete` | HUD binds to this to show "ready" indicator |

---

### Step-by-Step Build

#### Step 1 — Create the Component

1. Navigate to `Content/ProjectAnomaly/Player/Components/`
2. Right-click → **Blueprint Class** → **Actor Component** → name: `AC_UtilityComponent`
3. Double-click to open

#### Step 2 — Add Variables and Dispatchers

Add all 4 variables from the table above.

For `bCanThrow`:
- Name: `bCanThrow`, Type: Bool, Default: **checked** (true), Instance Editable: off

For `ThrowForce`:
- Name: `ThrowForce`, Type: Float, Default: `2000.0`, Instance Editable: **on**

For `CooldownDuration`:
- Name: `CooldownDuration`, Type: Float, Default: `45.0`, Instance Editable: **on**

For `CooldownTimerHandle`:
- Name: `CooldownTimerHandle`, Type: **Timer Handle** (search for it), no default needed

Add Event Dispatchers:
- My Blueprint → Event Dispatchers → **+** → name: `OnSatchelThrown`
- **+** again → name: `OnCooldownComplete`

#### Step 3 — Build the ThrowSatchel Function

My Blueprint → Functions → **+** → name: `ThrowSatchel`

Build this inside the function graph:

---

**Node 1: Gate on bCanThrow**
- Right-click → `Branch`
- Connect `bCanThrow` variable to Condition
- Connect function entry exec → Branch

**Node 2: Early exit if can't throw**
- From Branch **False** → `Return Node`

**Node 3: Get the character owner**
- From Branch **True** → `Get Owner` → `Cast to Character`
- Connect True exec → Cast to Character

**Node 4: Get the arms mesh component**
- From `As Character` → drag → search `Get Mesh` → add it
- This returns the character's skeletal mesh component (the FPS arms in the FPS template)

**Node 5: Get Socket Location and Rotation**

For location:
- From Get Mesh output → drag → search `Get Socket Location` → add it
- **In Socket Name**: type `GripPoint` (this is the FPS template's weapon socket)
  > If your character mesh doesn't have `GripPoint`, type whatever the socket is named on your character. To find it: open `BP_FirstPersonCharacter` → Mesh component → Skeleton → look at Socket list.
- This gives you a Vector for where to spawn the satchel

For rotation:
- From Get Mesh → `Get Socket Rotation` → Socket Name: `GripPoint`
- This gives you a Rotator

**Node 6: Make a Spawn Transform**
- Right-click → search `Make Transform` → add it
- **Location**: connect Get Socket Location output
- **Rotation**: connect Get Socket Rotation output
- **Scale**: leave as default (1, 1, 1)

**Node 7: Spawn the Satchel**
- Right-click → search `Spawn Actor from Class` → add it
- **Class**: click the dropdown → search `BP_SatchelCharge` → select it
- **Spawn Transform**: connect Make Transform output
- **Collision Handling Override**: click dropdown → `Always Spawn, Ignore Collisions`
- Connect execution: Cast to Character success → Spawn Actor from Class

**Node 8: Set the InstigatorRef on the Spawned Satchel**
- From `Spawn Actor from Class`'s **Return Value** pin → drag → search `Set Instigator Ref` (or just `InstigatorRef`) → look for a **Set** node for `InstigatorRef`
  
  The trick: `Return Value` is a `BP_SatchelCharge` reference. Drag from it → type `InstigatorRef` → you'll see `Set InstigatorRef` — add it.
- **Value** to set: `Get Owner` (drag from My Blueprint or right-click → Get Owner) → connects the player character as the instigator
- Connect execution: Spawn Actor → SET InstigatorRef

**Node 9: Add Impulse (Throw Physics)**

The satchel needs to be physically thrown forward.

Get camera forward direction:
- Right-click → `Get Owner` → `Cast to Character` → `Get Controller` → `Cast to PlayerController` → `Get Player View Point`
- From `Get Player View Point`'s **Rotation** output → `Get Forward Vector`

Compute impulse vector:
- Right-click → `Vector × Float` multiply node
- Vector input: Get Forward Vector output
- Float input: `ThrowForce` variable
- Output: the impulse vector

Apply it:
- From the Spawn Actor Return Value → drag → search `Get Static Mesh Component` — wait, the satchel uses a Sphere Collision as root.
- Instead: drag from Return Value → `Get SatchelCollision` (the component you named in Section 10)

Actually, since the root component simulates physics, you can add impulse directly to the satchel actor's root:

- From Return Value (BP_SatchelCharge) → drag → search `Add Impulse` ... but this is on a PrimitiveComponent
- Better approach: From Return Value → drag → search `Get Root Component` → then `Cast to Primitive Component` → then `Add Impulse`
  - **Impulse**: connect the ThrowForce × ForwardVector vector
  - **Vel Change**: leave unchecked (false) for physics impulse

Alternatively, get the SatchelCollision directly:
- From Return Value → drag → search `Get Satchel Collision` (typed name of your component)

Connect execution: SET InstigatorRef → Add Impulse

**Node 10: SET bCanThrow = false**
- After Add Impulse → `SET bCanThrow` → uncheck the bool (set to false)
- Connect execution: Add Impulse → SET bCanThrow

**Node 11: Start the Cooldown Timer**
- Right-click → `Set Timer by Event` → add it
  
  For the Event pin, you need to bind it to the StartCooldown event:
  - Right-click in graph → **Add Custom Event** → name it `StartCooldown`
  - This creates a `StartCooldown` event node in the graph
  - Now right-click → `Create Event` → it asks for a function/event name → point it at StartCooldown
  
  Simpler approach: use `Set Timer by Function Name`:
  - Right-click → `Set Timer by Function Name`
  - Object: `Self`
  - Function Name: `StartCooldown` (exact string)
  - Time: `CooldownDuration` variable
  - Looping: false
  
- From Return Value of Set Timer → connect to SET CooldownTimerHandle
- Connect execution: SET bCanThrow → Set Timer

**Node 12: Call OnSatchelThrown Dispatcher**
- Drag `OnSatchelThrown` from Event Dispatchers → release → **Call**
- Connect execution: Set Timer → Call OnSatchelThrown

**Node 13: Return Node**
- Connect after Call OnSatchelThrown → Return Node

---

#### Step 4 — Build the StartCooldown Custom Event

The `StartCooldown` node is already in your graph from the previous step. Add two nodes coming from its exec pin:

1. **SET bCanThrow = true** (drag bCanThrow → Set → check the bool ON)
2. **Call OnCooldownComplete** (drag dispatcher → Call)

That's it. Just those two nodes.

#### Step 5 — Compile and Save

Click **Compile** → green checkmark → **Save**.

> **🔧 DEBUG — Test ThrowSatchel:**
> 1. Add `Print String "THROW CALLED"` at the very start of ThrowSatchel (before the Branch)
> 2. Add `Print String "SATCHEL SPAWNED"` after Spawn Actor
> 3. Wire input and play (Section 12 below)
> 4. Press Q → see both prints → after 2 seconds hear/see detonation
> 5. Press Q again immediately → nothing should happen (cooldown active)
> 6. If satchel spawns but doesn't fly: check Add Impulse is targeting the right component and ThrowForce is 2000+

---

## SECTION 12 — WIRING BP_FIRSTPERSONCHARACTER

**You are ADDING components and input events to the existing FPS character. Do not delete any existing nodes.**

> **⚠️ STOP — Read this before opening BP_FirstPersonCharacter:**
> The FPS template already has movement, jump, look, and fire wired in this Blueprint. Do NOT remove or disconnect any existing nodes. You are only ADDING:
> - Two new components in the Components panel
> - Two new input event nodes in the Event Graph

---

### Step 1 — Open the Character Blueprint

Navigate to `Content/FirstPerson/Blueprints/` → double-click `BP_FirstPersonCharacter`

### Step 2 — Add Your Components

In the **Components panel** (top-left):

1. Click **+ Add** → type `AC_MovementExtComponent` → select it → it appears in the list
2. Rename it `MovementExtComponent` (right-click → Rename)
3. Click **+ Add** again → type `AC_UtilityComponent` → select it
4. Rename it `UtilityComponent`

> **✅ CHECK:** You should now see `MovementExtComponent` and `UtilityComponent` in the Components list alongside the existing capsule, mesh, camera, etc.

### Step 3 — Verify Enhanced Input is Already Wired

Find the **Event BeginPlay** in the Event Graph. It should have a `Add Mapping Context` node somewhere connected to it, with `IMC_Default` set. The FPS template sets this up by default.

If you see it — you are good. Your new IA_ assets in IMC_Default will automatically work.

If you DO NOT see `Add Mapping Context` in BeginPlay: you need to add it.
- From BeginPlay → right-click → `Get Enhanced Input Local Player Subsystem`
  - Input: right-click → `Get Player Controller` (index 0)
- From Subsystem → `Add Mapping Context`
  - **Mapping Context**: `IMC_Default` (the FPS template's one in `Content/FirstPerson/Input/`)
  - **Priority**: `0`
- Connect BeginPlay → Add Mapping Context

### Step 4 — Wire IA_Grapple to TryGrapple

In the Event Graph, find an empty area away from existing nodes.

1. Right-click in empty graph space → **Enhanced Action Events** section → search `IA_Grapple` → select it
   - You get a node with `Started`, `Ongoing`, `Triggered`, `Canceled`, `Completed` exec pins
2. From the **Started** exec pin → drag → search `Get MovementExtComponent` (your variable)
   - The component variable appears in the list — select it
3. From the `MovementExtComponent` pin → drag → search `Try Grapple` → select it
4. Connect the execution: `Started` exec → `Try Grapple`'s exec pin

### Step 5 — Wire IA_Utility to ThrowSatchel

Still in the Event Graph:

1. Right-click → Enhanced Action Events → search `IA_Utility` → add it
2. From `Started` exec pin → drag → `Get UtilityComponent`
3. From `UtilityComponent` → `Throw Satchel`
4. Connect `Started` → `Throw Satchel` exec

### Step 6 — Compile and Save

Click **Compile** → green checkmark → **Save**.

> **⚠️ STOP — If you get a "variable not found" error:** The component wasn't added correctly. Check the Components panel — the component must appear there before variables for it appear in the graph.

---

## SECTION 13 — FULL TEST CHECKLIST

**Run every test in order. Fix each failure before moving on.**

### Grapple Tests

| # | What to Test | How | Expected | Fix If Failing |
|---|---|---|---|---|
| G-01 | AC_MovementExtComponent compiles | Open it → Compile | Green checkmark | If error mentions BP_GrappleAnchor: open that first → Compile → re-open component → Compile |
| G-02 | IA_Grapple input fires | Add `Print String "E PRESSED"` at IA_Grapple Started → Play → press E | Prints "E PRESSED" | IMC_Default not registered: check Add Mapping Context is in BP_FirstPersonCharacter BeginPlay |
| G-03 | TryGrapple function is called | Add `Print String "GRAPPLE CALLED"` at TryGrapple entry → Play → press E | Prints "GRAPPLE CALLED" | MovementExtComponent not attached to character, or Started pin not connected |
| G-04 | Line trace fires visually | Add `Draw Debug Line`: Start=CameraLoc, End=CameraLoc+(Fwd×3000), Color=Red, Duration=2.0 → Play → press E → look around | See a red line shooting from camera | No line = component not on character or get owner returning null |
| G-05 | Grapple launches player to anchor | Aim at a placed BP_GrappleAnchor → press E | Player launches toward anchor | If line trace not hitting: check BP_GrappleAnchor collision is set, check anchor is in open air |
| G-06 | Grapple doesn't trigger on walls | Aim at a wall → press E | Nothing happens | Cast to BP_GrappleAnchor not filtering: make sure the Cast Fail path leads to Return Node |
| G-07 | Player moves in all directions | Use a high anchor diagonally above | Player moves both up and sideways toward it | If only horizontal: bZOverride is false on Launch Character |

### Satchel Tests

| # | What to Test | How | Expected | Fix If Failing |
|---|---|---|---|---|
| S-01 | BP_SatchelCharge compiles | Open it → Compile | Green checkmark | FS_DamageInfo missing or field names wrong → re-check Section 5 |
| S-02 | AC_UtilityComponent compiles | Open it → Compile | Green checkmark | BP_SatchelCharge class not found → make sure it compiled first |
| S-03 | Q press calls ThrowSatchel | Print "THROW CALLED" at start of ThrowSatchel → Play → press Q | Prints "THROW CALLED" | IMC_Default Q binding missing or IA_Utility Started not connected |
| S-04 | Satchel spawns and flies | Play → press Q → watch viewport | A small sphere appears and flies forward, arcs, bounces | Simulate Physics off → go back to Section 10 Step 2 and enable it |
| S-05 | Fuse fires after 2 seconds | Print "DETONATING" at start of Detonate event → Play → throw | Prints "DETONATING" ~2s after throw | Function Name in Set Timer doesn't match "Detonate" exactly |
| S-06 | Blast jump works | Stand within 600 units of satchel when it detonates | Player gets launched | bImpulseVelChange OFF: go to BP_SatchelCharge → BlastForce component → check bImpulseVelChange is ON |
| S-07 | Cooldown blocks second throw | Throw → immediately press Q again | Second throw does nothing | bCanThrow not being set to false: check the SET node is connected after Spawn Actor |
| S-08 | Cooldown resets after 45s | Throw → wait 45+ seconds → press Q | New satchel appears | StartCooldown event name doesn't match timer string: check spelling |

### Integration Tests

| # | Test | Expected |
|---|---|---|
| I-01 | Grapple while satchel is in flight | Both work independently with no interference |
| I-02 | Throw satchel → grapple away → blast catches you mid-flight | Blast jump works even while grappling |
| I-03 | Die (fall out of world or use console `kill`) and respawn | Cooldown resets, both mechanics work after respawn |

---

## SECTION 14 — KEY VALUES AT A GLANCE

All these are Instance Editable or directly accessible — no need to reopen Blueprint graphs to adjust them.

| Value | Default | Where to Change |
|---|---|---|
| GrappleSpeed | 2500.0 | Select `MovementExtComponent` on player → Details panel |
| GrappleRange | 3000.0 | Select `MovementExtComponent` on player → Details panel |
| ThrowForce | 2000.0 | Select `UtilityComponent` on player → Details panel |
| CooldownDuration | 45.0 | Select `UtilityComponent` on player → Details panel |
| BaseDamage (satchel) | 100.0 | Open `BP_SatchelCharge` → Variable defaults |
| BlastRadius | 600.0 | Open `BP_SatchelCharge` → Variable defaults |
| FuseDuration | 2.0 | Open `BP_SatchelCharge` → Variable defaults |
| Blast Force Strength | 200000.0 | Open `BP_SatchelCharge` → select `BlastForce` component → Details |
| Grapple Anchor sphere radius | 50.0 | Open `BP_GrappleAnchor` → select `GrappleSphere` → Details |

---

## SECTION 15 — GLOSSARY

| Term | What It Means |
|---|---|
| **Actor Component (AC_)** | Modular logic block attached to an Actor. Can't be placed in the world alone. Lives on an Actor host. |
| **Blueprint Actor (BP_)** | A full game object you can place in the world (e.g. BP_GrappleAnchor, BP_SatchelCharge) |
| **Line Trace** | UE's raycast. Fires an invisible ray and returns what the first hit is |
| **Launch Character** | Overrides the Character Movement Component's velocity to physically hurl the player |
| **Radial Force Component** | Pushes nearby physics objects outward in a sphere — perfect for explosions |
| **bImpulseVelChange** | Setting on Radial Force. When true: force is applied as a velocity change, ignoring mass. Heavy things and light things get thrown equally far. |
| **Event Dispatcher** | A broadcast. When you "Call" it, all bound functions fire simultaneously. One sender, many listeners. |
| **Custom Event** | A named entry point in a Blueprint graph. Timers call these by name string. |
| **Set Timer by Function Name** | Waits a given time, then calls a Custom Event by its string name. The string must match exactly. |
| **Cast to** | Checks if an object is a specific type. If yes: gives access to that type's variables and functions. If no: the "Cast Failed" path fires. |
| **Instance Editable** | Makes a variable editable per-placed-instance in the level editor — so you can set GrappleSpeed differently on different instances of the component. |
| **BPI_ (Blueprint Interface)** | A contract. "I implement BPI_Damageable" means "you can call ReceiveDamage on me without knowing my type." |
| **FS_ (Struct)** | A named bundle of variables. FS_DamageInfo passes all damage info as a single package. |
| **GripPoint** | Socket name on the FPS template's arm mesh where weapons attach. Used as the satchel spawn origin. |
| **PhysicsActor (Collision Preset)** | Makes a component participate in physics simulation — it gets pushed by forces and other objects |
| **Simulate Physics** | When enabled on a component, UE's physics engine takes over its movement (gravity, bouncing, etc.) |
| **Enhanced Input** | UE 5's input system. Input Actions (IA_) define what an action is. Input Mapping Contexts (IMC_) bind keys to actions. |

---

## APPENDIX — COMPILE ORDER CHECKLIST

Use this as a final check before testing in-game:

```
□  FS_DamageInfo compiled and saved
□  BPI_Damageable compiled and saved
□  IA_Grapple saved
□  IA_Utility saved
□  IMC_Default has E → IA_Grapple and Q → IA_Utility bindings
□  BP_GrappleAnchor compiled and saved (no errors)
□  BP_GrappleAnchor instances placed in the level (at least 3)
□  BP_SatchelCharge compiled and saved (no errors)
□  AC_MovementExtComponent compiled and saved (no errors)
□  AC_UtilityComponent compiled and saved (no errors)
□  BP_FirstPersonCharacter has MovementExtComponent and UtilityComponent in Components panel
□  BP_FirstPersonCharacter has IA_Grapple Started → TryGrapple wired
□  BP_FirstPersonCharacter has IA_Utility Started → ThrowSatchel wired
□  BP_FirstPersonCharacter compiled and saved
□  Level saved with BP_GrappleAnchor instances
```

---

## APPENDIX — WHAT TO TELL YOUR TEAMMATE

When handing off `AC_MovementExtComponent`:

- `TryDash()` is an **empty stub function** with one `Direction (Vector)` input — you left it empty intentionally
- The `bDashAvailable` variable exists and is set to `false` — they wire their rage system to set it `true` when `OnEnragedStateBegin` fires and `false` when `OnEnragedStateEnd` fires
- The `OnDashUsed` event dispatcher exists — they should call it when dash fires
- They also need to implement the body of `TryDash()`: check `bDashAvailable` → if true → `Launch Character (Direction × DashForce)` → `Call OnDashUsed`

---

*Guide written for: Project Anomaly — Vansh's Branch (UE 5.4.4)*
*Based on: GDD v1.1 · TDD v1.1 · Blueprint Guides P1/P2/P3 · Architecture Document · Dependency Map · FPS Template defaults*
*Starting point: Default UE5 First Person Template*
