# Battle of Spargus — Mod Readme

- **Game:** Jak 3
- **Repository:** [`whozghiar/jak3-mod-battle-of-spargus`](https://github.com/whozghiar/jak3-mod-battle-of-spargus)
- **Status:** In development, no release. Four scenarios are implemented; their in-game behavior is not verified (see [§4](#4-status-and-known-limits)).

This is the developer and agent documentation of the mod: what it changes, how it works, how
to test it, its limits, and its change log. Players read the root [`README.md`](../../../README.md).

**Contents:** [1. What the mod does](#1-what-the-mod-does) · [2. Architecture](#2-architecture) · [3. How to test](#3-how-to-test) · [4. Status and known limits](#4-status-and-known-limits) · [5. Change log](#5-change-log)

---

## 1. What the mod does

The mod turns Spargus (level `waswide`, DGO `WWD`) into a war zone. Spargus already runs Haven
City's traffic and citizen engine for its wastelanders; the mod ships the code and art of Haven
City's units inside WWD and drives their population through the traffic engine's per-type
counts. Five scenarios are available:

| Menu label | Mode (`*spargus-invasion-mode*`) | Freedom League guards and squads | Haven City civilians | Metal Heads (grunt, flitter, predator) | Wastelander militia | Who fights whom |
|---|---|---|---|---|---|---|
| Off (retail Spargus) | `'off` | no | no | no | retail | retail Spargus |
| Battle of Spargus | `'battle` | yes | no | no | yes | guards and militia, on sight |
| Pacified Spargus | `'pacified` | yes | yes | no | no | nobody: the guards patrol |
| Invasion: Freedom League defends | `'invasion-ff` | yes | no | yes | no | guards against Metal Heads; Metal Heads also go for Jak |
| Invasion: Wastelanders defend | `'invasion-wl` | no | no | yes | yes | militia against Metal Heads; Metal Heads also go for Jak |

### Switching it on

The mod registers one entry, `spargus-invasion`, in the retail Mods menu: open it with
**L3 + SELECT** (it works in a retail boot, see
[`mods_menu.md`](../guides/mods_menu.md)), then choose `spargus-invasion`.

| Submenu | Choices | Default | Variable (`goal_src/jak3/engine/ai/traffic-h.gc`) |
|---|---|---|---|
| Scenario | the five rows of the table above | Off (retail Spargus) | `*spargus-invasion-mode*` |
| Freedom League guards | 4 / 8 / 12 / 16 | 8 | `*spargus-invasion-guard-count*` |
| Guard squads | 0 / 1 / 2 / 3 | 2 | `*spargus-invasion-squad-count*` |
| Haven City civilians | 6 / 12 / 18 / 24 | 12 | `*spargus-invasion-civilian-count*` |
| Metal Heads | 5 / 10 / 15 / 20 | 10 | `*spargus-invasion-metalhead-count*` |

Each choice is a check-marked flag. Confirming one stores the value and calls
`*mod-spargus-apply-hook*`:

- **In Spargus** the change applies at once, with no reload: the types a scenario removes are
  sent back to the traffic pool, and the new units spawn over the next frames.
- **Anywhere else** the hook is a no-op: the choice is recorded and applied the next time
  Spargus activates.
- **Nothing is saved.** The five variables are plain `define`s, so every boot starts Off with
  the defaults above.

The menu file sits in GAME.CGO, not in WWD, on purpose: the Mods menu keeps a pointer to the
builder and calls it whenever it opens, anywhere in the game, so a builder living in WWD would
point into freed memory once Spargus unloads.

### What Off leaves stock

While the mode is `'off` and no scenario has been armed during the current Spargus visit,
`mod-spargus-apply` returns without touching the traffic engine, and every hook returns
immediately (`mod-spargus-invasion-active?` is false). The traffic engine keeps the settings
the retail traffic manager gave it. A build with the mod still differs from stock in these
ways, all inert for gameplay according to the code:

- WWD.DGO carries 18 extra code objects, 4 texture pages and 7 art groups, and its art-group
  block is re-sorted (the retail art groups keep their relative order). `waswide.fr3` holds 7
  extra merc models.
- `waswide-login` allocates one `ff-squad-control` and one `mh-squad-control` on the waswide
  loading-level heap and installs the six hooks, every visit.
- The shared unit code (`guard.gc`, `mh-squad-member.gc`, the three `metalhead-*.gc`,
  `wlander-male.gc`) gained one indirect hook call per call site and null tests that, per the
  in-code comments, always pass in Haven City. The hooks only point at the mod's functions
  between waswide login and waswide deactivate, and each target, init and trans hook first
  checks that a scenario is selected and `*city-mode*` is `'waswide`.

---

## 2. Architecture

### 2.1 Files added or changed

| File | Change | Why |
|---|---|---|
| `goal_src/jak3/engine/ai/traffic-h.gc` (GAME) | Adds `mod-spargus-hook-none`, the six hook variables (all set to it), `*spargus-invasion-mode*` (`'off`) and the four population counts. | The shared unit code is linked in Haven City (CWI) as well as in Spargus (WWD), so its call sites need symbols that resolve everywhere, including when the WWD-only mod file is not loaded. The no-op returns `object`, which fixes every hook's type so the mod's implementations (some return a process) can be assigned to them. |
| `goal_src/jak3/levels/wascity/mod-spargus-invasion.gc` (new, WWD) | All scenario logic: tuning constants, scenario predicates, traffic wiring, target acquisition, unit setup, level life cycle, REPL helper `mod-spargus-invasion-debug`. | Lives with the level it changes; only loaded while Spargus is. |
| `goal_src/jak3/pc/features/mod-spargus-invasion-menu.gc` (new, GAME) | Static popup-menu tree, `mod-spargus-invasion-set-mode`, `mod-spargus-invasion-reapply`, builder `mod-spargus-invasion-build-menu`, and the `mods-menu-register` call. | Must be resident (see §1). Fully static, so it costs no heap. |
| `goal_src/jak3/levels/wascity/waswide-init.gc` | `waswide-login` calls `mod-spargus-invasion-login` last; `waswide-activate` calls `mod-spargus-invasion-activate` last; `waswide-deactivate` calls `mod-spargus-invasion-deactivate` first. | Activate must run after `traffic-start`, which respawns the traffic manager and resets every count. Deactivate must run while `*city-mode*` is still `'waswide` and the traffic engine still exists. |
| `goal_src/jak3/levels/city/traffic/citizen/guard.gc` | Target hook first in `crimson-guard-method-290`; init hook at the tail of `citizen-method-194`. Null tests: `*cty-faction-manager*` in `enemy-common-post`; `attacker-info` in the `'member-attacked` event, `crimson-guard-method-290` and twice in `crimson-guard-method-261`; `*cty-attack-controller*` in `citizen-method-215`. | Spargus has no attack controller and no faction manager (see §2.5). In `citizen-method-215`, a nonzero test alone does not catch `#f` (the false symbol has a nonzero address), so the retail code went on to allocate an attacker from `#f`; the new test is the one `civilian::citizen-method-215` already uses. |
| `goal_src/jak3/levels/city/common/mh-squad-member.gc` | Init hook at the tail of `citizen-method-194`; target hook first in `get-current-enemy`. `enemy-common-post` keeps the current `faction-mode` when no faction manager exists. `attacker-info` null tests in `get-current-enemy` and `find-new-focus`. | Same reasons, for the three Metal Head species. |
| `goal_src/jak3/levels/city/traffic/citizen/metalhead-flitter.gc`, `metalhead-grunt.gc`, `metalhead-predator.gc` | `citizen-method-215` gets the same attack-controller test as the guard. | Without it, a Metal Head outside Haven City allocated an attacker from `#f` and crashed in `init!`. |
| `goal_src/jak3/levels/wascity/wlander-male.gc` | The `wlander` `active` state's trans calls `*mod-spargus-wlander-trans-hook*` just before its retail "focus found, go hostile" check. | Retail wastelanders only look for a target from the `panic` state; the hook lets them do it during ordinary city life in the scenarios where they fight. |
| `goal_src/jak3/dgos/wwd.gd` | 17 unit code objects after `traffic-manager.o`; `mod-spargus-invasion.o` after `wlander-female.o` and before `waswide-init.o`; 4 texture pages; 7 art groups; art-group block sorted by decreasing size. | See §2.2. |
| `goal_src/jak3/dgos/game.gd` | `mod-spargus-invasion-menu.o` right after `mods-menu.o`. | Menu file in GAME.CGO, after the registry it calls; `traffic-h.o`, which holds its variables, is earlier in the same list. |
| `decompiler/config/jak3/jak3_config.jsonc` | `extra_art_groups_by_dgo` entry for `WWD.DGO` (7 art groups with their home DGO). | Bakes the units' merc geometry into `waswide.fr3`, see §2.2. |

No `game.gp` change is needed: Jak 3's `cgo-file` (`goal_src/jak3/lib/project-lib.gp`) resolves
every `.o` listed in a `.gd` to the `.gc` of the same name found anywhere under `goal_src/jak3`
(`get-gsrc-path`, backed by a recursive scan in `goalc/make/MakeSystem.cpp`). A `.o` listed in
several `.gd` files is compiled once, at its first occurrence in `game.gp`'s `cgo-file` order:
the CWI unit files keep their CWI compile step, while `metalhead-grunt.o` and
`metalhead-predator.o` are now compiled in WWD's sequence, because `ctypesb.gd` is processed
after `wwd.gd`. The two new files are compiled at their position in `wwd.gd` and `game.gd`.

### 2.2 Code and art residency in WWD

**Code.** These 17 objects normally ship in CWI (`metalhead-grunt.o` and `metalhead-predator.o`
in CTYPESB); `wwd.gd` lists them in the same order as `cwi.gd` / `ctypesb.gd` so the link order
stays identical: `cty-guard-projectile`, `citizen-enemy`, `mh-squad-member-h`, `guard`,
`guard-grenade`, `guard-tazer`, `guard-rifle`, `guard-states`, `citizen-norm`, `citizen-fat`,
`citizen-chick`, `mh-squad-member`, `metalhead-flitter`, `metalhead-grunt`,
`metalhead-predator`, `ff-squad-control`, `mh-squad-control`. `mod-spargus-invasion.o` follows
every unit file and precedes `waswide-init.o`, which calls it.

**Art.** Adding an `-ag.go` to a `.gd` loads the art group (joints, animations), but the PC merc
renderer only draws models whose geometry is in the `.fr3` of a loaded level: without more, the
unit links and animates but draws nothing. `extra_art_groups_by_dgo` makes `task extract` bake
each listed model into `waswide.fr3`. The `:<HOME.DGO>` suffix names the level whose texture
remap table resolves the model's texture ids; a wrong home renders the model untextured. The
pattern is documented in the Lisp wiki,
[jak2 §3.5 Merc geometry & FR3 residency](../../../.agents/skills/goal-lisp/wiki/jak2.md#35-merc-geometry--fr3-residency);
the decompiler code is `decompiler/level_extractor/extract_level.cpp`.

| Art group | Home DGO (texture remap) | Size in WWD.DGO (bytes, from the `wwd.gd` comments) | Unit |
|---|---|---|---|
| `crimson-guard-ag` | CTYPESA | 288976 | Freedom League guard |
| `citizen-fat-ag` | CTYPEPA | 245568 | Haven City civilian |
| `citizen-norm-ag` | CTYPEPA | 236544 | Haven City civilian |
| `citizen-chick-ag` | CTYPEPA | 198400 | Haven City civilian |
| `predator-ag` | CTYPEPB | 188160 | Metal Head predator |
| `city-grunt-ag` | CTYPESB | 124816 | Metal Head grunt |
| `city-flitter-ag` | CTYPESB | 106112 | Metal Head flitter |

Together they add 1,388,576 bytes of art groups to WWD. The four texture pages come from the
same home levels: `tpage-957` (CTYPESA, guard), `tpage-1758` (CTYPESB, grunt and flitter),
`tpage-956` (CTYPEPA, civilians), `tpage-958` (CTYPEPB, predator).

> [!WARNING]
> **The art-group order in `wwd.gd` is load-bearing.** Once the first art group logs in
> (`art-group::relocate`, `goal_src/jak3/engine/anim/joint.gc`), the level switches to
> `tiny-edge` load buffers, and `load-buffer-resize` (`goal_src/jak3/engine/level/level.gc`)
> shrinks each of the two DGO buffers to the size of the object it just held plus 2 KB. The
> object loaded two slots later lands in that buffer, so no art group may be more than 2 KB
> larger than the one two positions before it. Keep the block sorted by decreasing size, as
> retail WWD is. Breaking the rule prints `dgo file header ... has overrun heap`, then fails
> the klink `version == 5` assert. A larger DGO buffer does not help: the resize overwrites it.

### 2.3 Hooks

Every hook takes the calling object and returns an object. The defaults (`mod-spargus-hook-none`)
return `#f`; a target hook that returns `#f` lets the retail code run.

| Hook variable | Called from | Mod implementation | What it does |
|---|---|---|---|
| `*mod-spargus-guard-target-hook*` | `guard.gc`, `crimson-guard-method-290` | `mod-spargus-guard-target` | Battle: nearest wastelander. Invasion FL: nearest Metal Head (`mh-squad-member`). Otherwise `#f`, which with no attack controller means no target: a Pacified patrol. |
| `*mod-spargus-guard-init-hook*` | `guard.gc`, tail of `citizen-method-194` | `mod-spargus-guard-init` | Battle and Invasion FL: one guard in three gets the grenade launcher, the others the rifle (no taser: a taser guard is melee-only and tries to arrest), and `sticky-weapon` stops `crimson-guard-method-270` from re-rolling it. Battle only: gives the guard back `process-mask enemy`. |
| `*mod-spargus-mh-target-hook*` | `mh-squad-member.gc`, `get-current-enemy` | `mod-spargus-mh-target` | Invasion FL: nearest guard or Jak. Invasion WL: nearest wastelander or Jak. |
| `*mod-spargus-mh-init-hook*` | `mh-squad-member.gc`, tail of `citizen-method-194` | `mod-spargus-mh-init` | Both invasions: gives the Metal Head `process-mask enemy`. |
| `*mod-spargus-wlander-trans-hook*` | `wlander-male.gc`, `wlander` `active` trans | `mod-spargus-wlander-trans` | Battle and Invasion WL: a wastelander with no focus calls the retail `search-for-focus` on its scan frame. |
| `*mod-spargus-apply-hook*` | the menu's `mod-spargus-invasion-set-mode` and `mod-spargus-invasion-reapply` | `mod-spargus-apply-hook-impl` | Re-applies the current scenario (`mod-spargus-apply`). |

**Why the enemy mask.** `citizen-init-by-other` (`citizen.gc`) clears `process-mask enemy` on
every citizen, and retail `wlander::search-for-focus` only accepts processes that carry it. That
is why retail wastelanders never notice guards. The init hooks put the mask back on exactly the
units the militia must fight. In the invasion the guards do not get it: per the code comment, it
would make Jak's auto-aim lock onto his allies.

**Target search.** `mod-spargus-find-foe` runs the same 60 m box query as
`wlander::search-for-focus` (`fill-actor-list-for-box` on `*actor-hash*`, 64 shapes) but keeps
the closest living foe instead of the first one, and optionally compares Jak's distance.
`mod-spargus-hold-or-scan` keeps the current foe while it lives and only queries on the unit's
scan frame: one frame in 16, staggered by `traffic-id`, the trick retail uses in
`crimson-guard-method-270`. Per the docstrings, a guard target returned from method 290 is
enough to drive the whole retail guard AI (`crimson-guard-method-261` copies it into
`target-status`, the `active` state goes hostile).

### 2.4 Level life cycle

1. **`waswide-login` → `mod-spargus-invasion-login`.** Sets `*mod-spargus-armed*` to `#f`,
   allocates and initialises the two squad controllers on the loading-level heap (writable only
   during login, so this happens even when the mod is Off), keeps them private in
   `*mod-spargus-ff-squad*` / `*mod-spargus-mh-squad*`, and points the six hooks at the mod's
   functions. Prints `mod-spargus-invasion-login` to the console.
2. **`waswide-activate` → `mod-spargus-invasion-activate`.** Runs after `traffic-start`. If a
   previous arming published the squads, unpublishes them; marks the visit unarmed; calls
   `mod-spargus-apply`.
3. **`mod-spargus-apply`.** Does nothing unless a scenario is active or was armed during this
   visit. When a scenario is active and the visit is not armed yet, `mod-spargus-arm-squads`
   publishes the squads into `*ff-squad-control*` / `*mh-squad-control*`, points them at the
   traffic engine and initialises them (`squad-control-method-10`). They are deliberately not
   registered with the engine: a registered squad runs its per-frame update and alert logic
   over the whole Spargus traffic, and `ff-squad-control` lists `wlander-male` as one of its
   guard types (guard type 3). Arming also clears the Freedom League squad's guard-type mask for
   `wlander-male`, so its alert logic never rewrites the wastelanders' target count.
   `mod-spargus-apply` then writes every type (§2.5).
4. **Menu change** → `*mod-spargus-apply-hook*` → `mod-spargus-apply`.
5. **`waswide-deactivate` → `mod-spargus-invasion-deactivate`.** If armed, applies the `'off`
   state once (foreign types parked, militia restored) and restores the chosen mode; points the
   hooks back at `mod-spargus-hook-none`; unpublishes and forgets the squads, whose storage dies
   with the level heap.

### 2.5 Traffic wiring, limits and real unit counts

**Why no Haven City controllers.** Per the header of `mod-spargus-invasion.gc`, Spargus
deliberately runs without `*cty-attack-controller*` and `*cty-faction-manager*`. The attack
controller gives one of its attacker slots to every civilian, and every wastelander is one
(`wlander` derives from `civilian`): it runs dry and `init!` asserts a few seconds into the city.
A live faction manager makes `traffic-manager-method-22` rewrite the counts every frame,
overriding the mod's. Targets therefore come from the hooks, and the dereferences of these two
controllers (and of `attacker-info`) listed in §2.1 are null-guarded.

**`mod-spargus-set-type`** points one traffic type at a level and writes, by hand, what the
retail engine only computes when the traffic manager starts: the level (in the static
`*traffic-info*` `traffic-object-levels`, which outlives the level, and in the type info), the
`want-count` (pool size), the `target-count` (active units the spawner aims for) and the
`ttf3` flag. Parking a type sets level `#f`, both counts to 0, clears `ttf3` and sends
`'traffic-off-force` to every live unit of that type.

**Pool cap.** `reset-and-init-from-manager` (`goal_src/jak3/levels/city/traffic/traffic-engine.gc`)
gives each traffic type a 20-handle slice of `inactive-object-array`, and `add-reserved-process`
silently drops a 21st process. A want-count above 20 makes the spawner create a new untracked
unit every frame until memory runs out. Every count the mod writes is clamped to 20.

| Constant (`mod-spargus-invasion.gc`) | Value | Role |
|---|---|---|
| `SPARGUS_INVASION_ENGAGE_DIST` | 60 m | Foe search radius, same as `wlander::search-for-focus` |
| `SPARGUS_INVASION_SCAN_MASK` | 15 | One foe search every 16 frames per unit |
| `SPARGUS_INVASION_MILITIA_COUNT` | 14 | Retail wastelander want-count per sex (`traffic-manager-method-21`); target 13 |
| `SPARGUS_INVASION_SQUAD_SIZE` | 5 | Guards one squad (`formation`) reserves in the inactive guard pool |
| `SPARGUS_INVASION_MAX_POOL` | 20 | Per-type cap |

**Guards.** Squads take their slots in the `guard-a` pool first (5 each, at most 4 squads), and
loose guards get what is left of the 20. `guard-a` gets want = loose + 5 × squads and
target = loose; `formation` gets want = target = squads. Loose guards actually spawned:

| Guard squads \ Freedom League guards | 4 | 8 | 12 | 16 |
|---|---|---|---|---|
| 0 | 4 | 8 | 12 | 16 |
| 1 | 4 | 8 | 12 | 15 |
| 2 (default) | 4 | 8 (default) | 10 | 10 |
| 3 | 4 | 5 | 5 | 5 |

**Civilians and Metal Heads.** One civilian in six is `civilian-fat`, the rest split between
`civilian-male` and `civilian-female`; one Metal Head in five is a predator, the rest split
between grunts and flitters. Want and target are equal:

| Civilians | male / female / fat | Metal Heads | grunt / flitter / predator |
|---|---|---|---|
| 6 | 3 / 2 / 1 | 5 | 2 / 2 / 1 |
| 12 (default) | 5 / 5 / 2 | 10 (default) | 4 / 4 / 2 |
| 18 | 8 / 7 / 3 | 15 | 6 / 6 / 3 |
| 24 | 10 / 10 / 4 | 20 | 8 / 8 / 4 |

**Militia.** `wlander-male` and `wlander-female` keep the retail want 14 / target 13 except in
Pacified and Invasion FL, where they are parked.

### 2.6 Memory

- The two squad controllers live on the waswide loading-level heap, allocated at every login
  and forgotten at deactivate.
- WWD grows by the art groups above (about 1.39 MB), four texture pages and 18 code objects.
- The menu is static data in GAME.CGO and allocates nothing.

### 2.7 Design history

Per the header of `mod-spargus-invasion.gc`, a first rework streamed Haven City's ctype levels
into waswide borrow slots instead of shipping the art in WWD. It loaded and spawned correctly,
but the game crashed with a Windows heap failure a few seconds after any ctype level was hosted
by Spargus, even with no unit spawned. The mod went back to shipping the art in WWD, the
approach of the first playable version.

---

## 3. How to test

1. Select the game:

   ```bash
   task set-game-jak3
   ```

2. The mod changes no C++. The decompiler must include master-dev's `extra_art_groups_by_dgo`
   support (`decompiler/level_extractor/extract_level.cpp`); rebuild it with
   `task build-release-decomp` only if your binary predates it.
3. Re-extract, so `waswide.fr3` gets the seven unit models (`(mi)` never regenerates a `.fr3`):

   ```bash
   task extract
   ```

   The log should show `extra_art_groups_by_dgo: baking '<art group>' into WWD.DGO (.fr3)` and
   `textures remapped via <HOME>.DGO` for each of the seven art groups.
4. Compile: `task repl`, then `(mi)`; or headless, `task compile-check`.
5. Boot cold: `task boot-game-retail` (proves the menu works without the debug segment) or
   `task boot-game`. A cold boot restores save slot 1, so use a save from which Spargus is
   reachable.
6. Enter Spargus with the mod Off: only wastelanders, as in stock Jak 3. The console prints
   `mod-spargus-invasion-login` when waswide logs in.
7. Open the Mods menu (L3 + SELECT) > `spargus-invasion` > `Scenario`, and try each scenario:

   | Scenario | What to look for |
   |---|---|
   | Battle of Spargus | Guards with rifles or grenade launchers fight the militia on sight; squads of guards move in formation. |
   | Pacified Spargus | No wastelanders; guards and squads patrol without fighting; Haven City civilians walk the streets. |
   | Invasion: Freedom League defends | No wastelanders; guards fight grunts, flitters and predators; Metal Heads also attack Jak. |
   | Invasion: Wastelanders defend | No guards; the militia fights the Metal Heads; Metal Heads also attack Jak. |

8. Change the counts in the four population submenus: the population follows within a few
   frames, and the loose guards follow the table in §2.5.
9. Back to Off inside Spargus: the foreign units disappear and the militia returns.
10. Leave Spargus for Haven City: the city must behave as in stock Jak 3.

With the REPL connected to the running game, `(mod-spargus-invasion-debug)` prints the mode,
`*city-mode*`, the armed flag, and for traffic types 0 to 13 their level, want, target, active
and inactive counts.

### Troubleshooting

| Symptom | Cause |
|---|---|
| Units fight or move but are invisible | `task extract` not run after a change to `extra_art_groups_by_dgo` |
| A unit model is untextured | Wrong `:<HOME.DGO>` in `extra_art_groups_by_dgo` |
| `dgo file header ... has overrun heap`, then a klink `version == 5` assert while loading Spargus | The `wwd.gd` art-group block is no longer sorted by decreasing size (§2.2) |
| Crash shortly after raising a count | A want-count above 20 for one type; every count must go through `mod-spargus-set-type`, which clamps it |
| Crash in unit code in Spargus after a code change | Something built `*cty-attack-controller*` in Spargus (its slots run dry and `init!` asserts), or new code dereferences `attacker-info`, `*cty-attack-controller*` or `*cty-faction-manager*` without a null test (§2.5) |

---

## 4. Status and known limits

- **Not verified in game.** The commit history records crash fixes found during development
  (DGO overrun, pool leak, scan cost), but no commit records a successful play test of the four
  scenarios, and this document was written from the code without building it.
- **No release.** Neither `whozghiar/jak-project` nor this repository has a release of the mod,
  and no launcher catalog lists it.
- **Loose guards shrink with squads.** All guards share the 20-slot `guard-a` pool, so 16 guards
  with 3 squads gives 5 loose guards (§2.5).
- **No Haven City controllers.** Without an attack controller or faction manager, units only
  engage what the 60 m nearest-foe search finds, every 16 frames. The squads are not registered
  with the traffic engine, so their per-frame update and alert logic does not run.
- **Guards never target Jak through the mod.** The guard target hook only looks for
  wastelanders or Metal Heads; the Metal Head hook includes Jak.
- **Jak's auto-aim in Battle.** Guards carry `process-mask enemy` in Battle so the militia can
  see them; the code comments state that this also exposes them to Jak's auto-aim.
- **Off after arming.** Switching back to Off inside Spargus parks the foreign types and
  restores the militia, but the squad controllers stay published in `*ff-squad-control*` /
  `*mh-squad-control*` until Spargus deactivates.
- **Settings are not saved.** Scenario and counts reset to Off and the defaults at every boot.
- **Naming.** The menu label `spargus-invasion` and the `mod-spargus-*` / `*spargus-invasion-*`
  symbols predate the repository; [`mods_menu.md`](../guides/mods_menu.md) expects the label to
  be the catalog key `battle-of-spargus` and the builder to be `mod-battle-of-spargus-build-menu`.
- **Stale comments.** The header of `mod-spargus-invasion.gc` still names the branch
  `jak3/features/battle_of_spargus` and refers to "the Tier-2 doc" (this file, §2.7), and the
  docstring of `mod-spargus-invasion-set-mode` still mentions borrow levels from the abandoned
  rework.

---

## 5. Change log

Oldest first. Merges of master-dev (`chore(sync)` commits) are left out.

| Date | Touched/Created Files | Technical Description | Objective |
|---|---|---|---|
| 2026-09-23 | `README.md` | `2efab3ef2`: player README created from the mod README template, still with its placeholders. | Open the `jak3/features/battle_of_spargus` branch. |
| 2026-09-23 | `traffic-h.gc`<br>`mod-spargus-invasion.gc` (new)<br>`mod-spargus-invasion-menu.gc` (new)<br>`waswide-init.gc`<br>`guard.gc`<br>`mh-squad-member.gc`<br>`metalhead-flitter.gc` / `metalhead-grunt.gc` / `metalhead-predator.gc`<br>`wlander-male.gc`<br>`wwd.gd` / `game.gd`<br>`jak3_config.jsonc` | `dcbf38352`: the whole mod. Four scenarios plus Off and four population counts in the Mods menu (GAME.CGO menu file); Haven City unit code (guards, civilians, three Metal Head species, both squad controllers) and art (4 texture pages, 7 art groups) shipped in WWD, merc geometry baked into `waswide.fr3` through `extra_art_groups_by_dgo`; WWD art groups sorted by decreasing size; resident no-op hooks in `traffic-h.gc`, installed at waswide login and restored at deactivate; attack-controller and faction-manager dereferences null-guarded; squads published and types rewritten only once a scenario is armed; Freedom League squad never registered with the Spargus traffic engine; every per-type pool capped at 20. | Turn Spargus into a war zone from the Mods menu while leaving it stock when Off; fix the DGO overrun and the pool leak found on the way. |
| 2026-09-23 | `mod-spargus-invasion.gc` | `95560dfbc`: `mod-spargus-hold-or-scan` now queries the actor hash only on the unit's scan frame, even when it holds no foe. Before, an idle unit ran a 60 m box query every frame. | Stop a full battle from slowing the game to a crawl. |
| 2026-10-02 | `README.md`<br>`index.json` (deleted) | `970c28280` (after the master-dev merge `7bb4b27c8`): the mod moved to its own repository, `whozghiar/jak3-mod-battle-of-spargus`. The README names the repository and notes the move; the branch's copy of the global catalog was removed. | One repository per mod. |
| 2026-10-02 | `README.md` | `b2b3d008b`: compliance checklist removed from the player README. | The player README is for players; the change log lives here. |
| 2026-10-02 | `docs/modding/current_mod/battle_of_spargus_readme.md` (new)<br>`README.md` | Technical README written from the code. Player README: the template placeholders (overview, features, build and extraction status, video) filled from the code, the catalog URL and the releases link point at this repository, the badge of the deleted `branch-sync-check.yaml` workflow is gone. | Document the mod for developers, agents and players, as the project rules require. |
