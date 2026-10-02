# Battle of Spargus — Jak 3

<p align="center">
  <img src="https://img.shields.io/badge/OpenGOAL-Mod-blue.svg" alt="OpenGOAL Mod">
  <img src="https://img.shields.io/badge/Game-Jak%203-orange.svg" alt="Target Game">
  <img src="https://img.shields.io/badge/AI--assisted-Modding-purple.svg" alt="AI Assisted">
</p>

> **Contents:** [Overview](#overview) · [Key Features](#key-features) · [Download & Play](#download--play-via-opengoal-launcher-players) · [Developer Setup](#developer-setup--local-compilation) · [Demo Video](#demonstration-video) · [Technical Documentation](#technical-documentation)

---

> [!NOTE]
> This mod moved from the `jak3/features/battle_of_spargus` branch of [whozghiar/jak-project](https://github.com/whozghiar/jak-project) to this repository. It has no release yet.

## Overview
Turns Spargus into a war zone: Freedom League guards, Haven City civilians, Metal Heads and the wastelander militia, in four scenarios picked from the Mods menu. Off by default, Spargus stays retail.

> [!NOTE]
> In development: the four scenarios are implemented, but their behavior in game is not verified yet (see the [technical documentation](docs/modding/current_mod/battle_of_spargus_readme.md)).

- **Target Game:** Jak 3
- **Repository:** [`whozghiar/jak3-mod-battle-of-spargus`](https://github.com/whozghiar/jak3-mod-battle-of-spargus)

## Key Features
- **Battle of Spargus:** Freedom League guards and the wastelander militia fight on sight.
- **Pacified Spargus:** guards patrol among Haven City civilians, and nobody fights.
- **Invasion, Freedom League defends:** guards against Metal Heads (grunts, flitters, predators), who also go for Jak.
- **Invasion, Wastelanders defend:** the militia against the Metal Heads, who also go for Jak.
- **Mods menu** (L3 + SELECT, Mods, `spargus-invasion`): the scenario, off by default, and the number of guards, guard squads, civilians and Metal Heads. A choice applies at once in Spargus; nothing is saved between boots.

## Download & Play via OpenGOAL Launcher (Players)

> [!TIP]
> **No developer environment required!** Players can install and play this mod directly using the official OpenGOAL Launcher:

### Option A — Add Custom Mod Source (Recommended)
1. In the **OpenGOAL Launcher**, navigate to **Settings ▸ Mods ▸ Add Custom Mod Source**.
2. Paste this catalog URL:
   ```text
   https://raw.githubusercontent.com/whozghiar/jak3-mod-battle-of-spargus/main/index.json
   ```
3. Go to the **Mods** tab, locate **Battle of Spargus**, and click **Install**.
4. Select your clean PS2 game ISO when prompted. The launcher will automatically extract assets and launch the game!

### Option B — Manual Installation from GitHub Releases
1. Download the pre-built package for your operating system from the [Releases](https://github.com/whozghiar/jak3-mod-battle-of-spargus/releases) tab (`windows-v*.zip` or `linux-v*.zip`).
2. Extract the archive into your OpenGOAL Launcher features directory:
   - **Windows:** `%APPDATA%\OpenGOAL-Launcher\features\jak3\mods\_local\battle-of-spargus\`
   - **Linux:** `~/.config/OpenGOAL-Launcher/features/jak3/mods/_local/battle-of-spargus/`
3. Launch the game from the OpenGOAL Launcher.

---

## Developer Setup & Local Compilation

If you want to modify or compile this mod locally from source:

### 1. Select the Active Game
Make sure your environment is targeting Jak 3:
```bash
task set-game-jak3
```

### 2. Binary Compilation
- **Status:** Not required (GOAL-only mod, standard binaries sufficient).
- **Details:** the decompiler change is configuration only (`extra_art_groups_by_dgo` in `decompiler/config/jak3/jak3_config.jsonc`), read when the decompiler runs: no rebuild, but step 3 is mandatory.

### 3. Asset Extraction
- **Status:** Required (`task extract`).
- **Details:** bakes the Haven City units the mod brings to Spargus (Crimson Guard, grunt, flitter, predator, citizens) into `WWD.fr3`. Without it they spawn but draw nothing.
```bash
task extract
```

### 4. Launch the Game
Run the game natively:
```bash
task boot-game
```
*(Or launch via the OpenGOAL REPL using `task repl`, then compile and run with `(mi)` and `(r)`).*

## Demonstration Video

No demonstration video yet.

## Technical Documentation
For the complete technical breakdown, architecture, and developer notes, refer to:
- [`docs/modding/current_mod/battle_of_spargus_readme.md`](docs/modding/current_mod/battle_of_spargus_readme.md)

---
*(AI-assisted)*
