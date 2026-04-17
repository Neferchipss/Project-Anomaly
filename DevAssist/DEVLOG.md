# Project Anomaly — Development Log
> GDD v1.1 · TDD v1.1 · UE 5.4 · Vertical Slice Target

---

## Quick Reference

| Doc | File |
|---|---|
| Architecture / Folder Structure | `ProjectAnomaly_UE5_Architecture.html` |
| Dependency Map + VS Checklist | `PA_DependencyMap.html` |
| Blueprint Guide Part 1 | `PA_Blueprint_Guide.pdf` |
| Blueprint Guide Part 2 | `PA_Blueprint_Guide_Part2.pdf` |
| Blueprint Guide Part 3 | `PA_Blueprint_Guide_Part3.pdf` |

**Content root:** `Content/ProjectAnomaly/`  
**Save slot:** `SaveSlot_01` → `Saved/SaveGames/SaveSlot_01.sav`

---

## Current Status

| Phase | Status | Owner | Notes |
|---|---|---|---|
| Phase 01 — Core Damage Loop | `NOT STARTED` | — | Week 1–2 |
| Phase 02 — Rage Loop + All Weapons | `NOT STARTED` | — | Week 3–4 |
| Phase 03 — Movement + Utility + AI | `NOT STARTED` | — | Week 5–7 |
| Phase 04 — Animation + Audio + UI | `NOT STARTED` | — | Week 8–10 |

**Status values:** `NOT STARTED` · `IN PROGRESS` · `BLOCKED` · `IN REVIEW` · `DONE`

---

## Phase 01 — Core Damage Loop
> Target: enemy takes damage from player and dies. Nothing else.

### Systems
| System | Status | Owner | Notes |
|---|---|---|---|
| FS_DamageInfo | `NOT STARTED` | — | Build day 1 — everything depends on this |
| All BPI_ Interfaces | `NOT STARTED` | — | Build day 1 — build all 6 at once |
| All Structs (FS_) | `NOT STARTED` | — | FS_WeaponRow, FS_EnemyRow, FS_SkillRow, FS_WaveRow |
| DT_Weapons (Pistol row) | `NOT STARTED` | — | |
| DT_Enemies (Brute row) | `NOT STARTED` | — | |
| IMC_Default + IA_Fire / IA_Move / IA_Look | `NOT STARTED` | — | |
| AC_HealthComponent | `NOT STARTED` | — | Needs FS_DamageInfo, BPI_Damageable |
| AC_EnemyHealthComponent | `NOT STARTED` | — | Needs FS_DamageInfo, BPI_Damageable |
| BP_WeaponBase | `NOT STARTED` | — | Abstract — do not place directly |
| BP_PlayerController | `NOT STARTED` | — | Must call Add Mapping Context in BeginPlay |
| BP_Weapon_Pistol | `NOT STARTED` | — | Hitscan; parry ammo gen on fire |
| AC_WeaponComponent | `NOT STARTED` | — | 3-slot inventory |
| BP_EnemyBase | `NOT STARTED` | — | Abstract — implements BPI_Damageable + BPI_WeakPointTarget |
| BP_Enemy_Brute (static, no AI) | `NOT STARTED` | — | AI added in Phase 03 |
| BP_PlayerCharacter | `NOT STARTED` | — | Wire all AC_ components in BeginPlay |

### Exit Criteria
- [ ] Player can move and aim
- [ ] Pistol fires hitscan
- [ ] Brute takes damage and dies
- [ ] Death removes enemy from scene
- [ ] No crashes on repeated fire

---

## Phase 02 — Rage Loop + All Weapons
> Target: hitting WPs fills rage, triggers Enraged State with all 4 bonuses.

### Systems
| System | Status | Owner | Notes |
|---|---|---|---|
| AC_RageComponent | `NOT STARTED` | — | Needs BPI_Rageable — blocks 6 downstream systems |
| BP_WeakPointActor | `NOT STARTED` | — | Spawnable collision volume, child of enemy socket |
| AC_WeakPointComponent | `NOT STARTED` | — | Needs BP_WeakPointActor, BPI_WeakPointTarget, AC_RageComponent |
| BP_Projectile_Base | `NOT STARTED` | — | Abstract — build before BP_Projectile_Enemy |
| BP_Projectile_Enemy | `NOT STARTED` | — | Needs BP_Projectile_Base |
| AC_ParryComponent | `NOT STARTED` | — | Build before BP_ParryWindow |
| BP_ParryWindow | `NOT STARTED` | — | Needs AC_ParryComponent |
| AC_StunComponent | `NOT STARTED` | — | Needs AC_EnemyHealthComponent |
| BP_Weapon_Shotgun | `NOT STARTED` | — | 8-pellet spread, penetrates up to 3 targets |
| BP_Weapon_AssaultRifle | `NOT STARTED` | — | Wind-up fire rate, movement penalty |
| DT_Weapons (Shotgun + AR rows) | `NOT STARTED` | — | |
| IA_Parry added to IMC_Default | `NOT STARTED` | — | |

### Exit Criteria
- [ ] WP hits add rage visibly
- [ ] Meter full → Enraged State starts
- [ ] All 4 Enraged abilities work
- [ ] State ends after ~1.5s
- [ ] Shotgun spread + AR wind-up correct
- [ ] Parry deflects green projectiles

---

## Phase 03 — Movement + Utility + All Enemy AI
> Target: full combat arena with all 5 enemy types behaving correctly.

### Systems
| System | Status | Owner | Notes |
|---|---|---|---|
| BP_GrappleAnchor | `NOT STARTED` | — | Build before AC_MovementExtComponent |
| AC_MovementExtComponent | `NOT STARTED` | — | Needs BP_GrappleAnchor, AC_RageComponent |
| BP_SatchelCharge | `NOT STARTED` | — | Build before AC_UtilityComponent |
| AC_UtilityComponent | `NOT STARTED` | — | Needs BP_SatchelCharge, BPI_Damageable |
| BP_SkillBase | `NOT STARTED` | — | Abstract — build before AC_SkillSlotComponent |
| BP_Save_Anomaly | `NOT STARTED` | — | Build before BP_GI_Anomaly |
| BP_Enemy_Brute (full AI) | `NOT STARTED` | — | Needs BP_EnemyBase complete |
| BP_Enemy_FlyingEye | `NOT STARTED` | — | EQS_FlyingEye_Strafe required |
| BP_Enemy_Spawner | `NOT STARTED` | — | Needs AC_SpawnDirectorComponent |
| BP_Enemy_FodderMelee | `NOT STARTED` | — | |
| BP_Enemy_FodderRanged | `NOT STARTED` | — | |
| All 5 AIC_ + BT_ + BB_ | `NOT STARTED` | — | Needs BP_EnemyBase, NavMesh |
| BTT_/BTS_/BTD_ shared nodes | `NOT STARTED` | — | All go in Enemies/Shared_AI/ |
| AC_SpawnDirectorComponent | `NOT STARTED` | — | Fodder spawn budget + timer |
| BP_EnemySpawner | `NOT STARTED` | — | Placed actor — build before BP_ArenaManager |
| BP_ArenaManager | `NOT STARTED` | — | Needs all enemy BPs, BP_EnemySpawner, DT_ArenaWaves |
| EQS_FlyingEye_Strafe | `NOT STARTED` | — | |
| EQS_Spawner_Retreat | `NOT STARTED` | — | |
| DT_Enemies (all 5 rows) | `NOT STARTED` | — | |
| DT_ArenaWaves (1 wave) | `NOT STARTED` | — | |
| DT_Skills (stub row) | `NOT STARTED` | — | Stub only — full registry post-VS |
| NavMesh | `NOT STARTED` | — | Press P in PIE to verify coverage |

### Exit Criteria
- [ ] Grapple pulls to anchor
- [ ] Dash works during enrage only
- [ ] Satchel detonates + blast jump works
- [ ] Brute engages, attacks, stuns
- [ ] FlyingEye strafes, fires, freezes on eye hit
- [ ] Spawner retreats and spawns fodder within budget
- [ ] Fodder melee charges, ranged keeps distance
- [ ] Arena wave clears, OnWaveComplete fires

---

## Phase 04 — Animation + Audio + UI + Polish
> Target: complete playable vertical slice — all systems integrated and presentable.

### Systems
| System | Status | Owner | Notes |
|---|---|---|---|
| AC_SkillSlotComponent + 1 skill BP | `NOT STARTED` | — | Needs BP_SkillBase, DT_Skills, AC_RageComponent events |
| BP_GI_Anomaly | `NOT STARTED` | — | Needs BP_Save_Anomaly |
| BP_SavePoint | `NOT STARTED` | — | Needs AC_SkillSlotComponent, BP_GI_Anomaly, BPI_Interactable |
| ABP_Player | `NOT STARTED` | — | Needs SK_Player, all player montages |
| ABP_Brute | `NOT STARTED` | — | |
| ABP_FlyingEye | `NOT STARTED` | — | |
| ABP_Spawner | `NOT STARTED` | — | |
| ABP_Fodder (shared) | `NOT STARTED` | — | |
| All BS_ locomotion blends | `NOT STARTED` | — | |
| Footstep System (AN_ notifies + DT_FootstepSurfaces) | `NOT STARTED` | — | 2 surfaces minimum for VS |
| WBP_HUD | `NOT STARTED` | — | Health, rage, ammo, skill slot, weapon wheel |
| WBP_SkillSelectMenu | `NOT STARTED` | — | Needs AC_SkillSlotComponent, BP_SavePoint |
| WBP_PauseMenu | `NOT STARTED` | — | Quit only for VS; settings stub |
| WBP_DeathScreen | `NOT STARTED` | — | 2.5s delay, reload level on respawn |
| PP_Enrage + PP_HitFlash | `NOT STARTED` | — | |
| NS_WeakPointHit + NS_SatchelExplosion | `NOT STARTED` | — | |
| BP_CameraShake_Hit | `NOT STARTED` | — | |
| All combat SC_ audio cues | `NOT STARTED` | — | Weapons, enemy alerts, satchel, impacts |
| SC_/SM_ sound class hierarchy | `NOT STARTED` | — | Master → SFX → UI |
| DT_FootstepSurfaces | `NOT STARTED` | — | |

### Exit Criteria
- [ ] All enemies animate correctly
- [ ] Footsteps play on correct surfaces
- [ ] All combat sounds present
- [ ] Screen effects on rage + damage
- [ ] Skill equip + activate works end to end
- [ ] Save point functional
- [ ] HUD reflects all live values
- [ ] Pause and death screens functional
- [ ] Full playthrough: spawn → fight → win → death → respawn

---

## Bug Tracker

| ID | Phase | System | Description | Severity | Status | Assigned | Fixed In |
|---|---|---|---|---|---|---|---|
| — | — | — | No bugs logged yet | — | — | — | — |

**Severity:** `CRIT` · `HIGH` · `MED` · `LOW`  
**Status:** `OPEN` · `IN PROGRESS` · `FIXED` · `WONT FIX`

### Bug Log Template
```
| BUG-001 | Ph01 | AC_HealthComponent | Describe the bug | CRIT | OPEN | Name | — |
```

---

## Known Risks & Blockers

| Risk | Impact | Mitigation |
|---|---|---|
| FS_DamageInfo not built day 1 | Every damage system blocked | Assign to one person, complete before anything else |
| BP_PlayerCharacter started before its AC_ components | Compile errors, wasted rework | AC_ components must be DONE before wiring BP_PlayerCharacter |
| NavMesh not covering full arena floor | All 5 enemy AI break silently | Check with P overlay every time geometry changes |
| BTT_ nodes missing Finish Execute on a code path | BT freezes, enemy gets stuck permanently | Use BP Guide Part 1 §17 checklist on every task node |
| Dissolve (MF_Dissolve) accidentally implemented in VS | Scope creep | Dissolve is explicitly post-VS — use Destroy Actor after ragdoll delay |
| AC_RageComponent delayed | Blocks Weapon boost, Dash, Parry, Skill, PP_Enrage, ABP_ rage state | Treat as critical path — start Phase 02 with this |

---

## Deferred Features (Post-VS)

These are explicitly out of scope. Do not implement during the vertical slice.

- Story / narrative / cutscenes
- Multiple levels, level streaming, 4-era hub
- Multiple Special Skills (only 1 needed)
- Spawner enemy full budget tuning (placeholder values OK)
- Foot IK (base locomotion only)
- L_MainMenu (start directly in arena)
- WBP_LoadingScreen
- Settings menu (pause has Quit only)
- In-game audio volume sliders
- Music tracks (SC_Music exists, no tracks)
- Post-process LUT per act
- Gamepad bindings
- 4-directional hit reaction variants (single montage only)
- MF_Dissolve death effect (ragdoll + destroy only)

---

## Architecture Rules (Team Reference)

### Naming Prefixes
| Prefix | Type |
|---|---|
| `BP_` | Blueprint Actor / Character / Pawn |
| `AC_` | Actor Component |
| `BPI_` | Blueprint Interface |
| `DA_` | Data Asset (Primary) |
| `DT_` | DataTable |
| `FS_` | Struct |
| `WBP_` | Widget Blueprint |
| `BT_` / `BB_` | Behavior Tree / Blackboard |
| `BTT_` / `BTS_` / `BTD_` | BT Task / Service / Decorator |
| `AIC_` | AI Controller |
| `ABP_` | Anim Blueprint |
| `AM_` | Anim Montage |
| `BS_` | Blend Space |
| `AN_` / `ANS_` | Anim Notify / Notify State |
| `NS_` | Niagara System |
| `M_` / `MI_` / `MF_` | Material / Instance / Function |
| `PP_` | Post Process Material |
| `SK_` / `SM_` | Skeletal / Static Mesh |
| `IA_` / `IMC_` | Input Action / Mapping Context |
| `SC_` / `SM_` | Sound Cue / Sound Mix |

### Key Rules
- **No hardcoded numbers in Blueprints** — all tunable values go in `Data/` via DT_ or DA_ assets
- **No direct class references between systems** — always go through a BPI_ interface
- **All data tables live in** `Content/ProjectAnomaly/Data/`
- **All structs live in** `Content/ProjectAnomaly/Structs/`
- **Shared AI BT nodes live in** `Content/ProjectAnomaly/Enemies/Shared_AI/`
- **Damage always routed through FS_DamageInfo** — never apply damage directly
- **BP_PlayerCharacter is built last in Phase 01** — all its AC_ components must exist first
- **BP_Projectile_Base must exist before BP_Projectile_Enemy**
- **BP_GrappleAnchor must exist before AC_MovementExtComponent**

---

## Team Notes

> Add dated entries below. Keep entries short — decisions, blockers, and context that isn't obvious from the code.

```
YYYY-MM-DD [Name]: Note here.
```

---
