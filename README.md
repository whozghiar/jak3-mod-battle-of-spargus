# Battle Of Spargus — Jak 3

<p align="center">
  <img src="https://img.shields.io/badge/OpenGOAL-Mod-blue.svg" alt="OpenGOAL Mod">
  <img src="https://img.shields.io/badge/Game-Jak%203-orange.svg" alt="Target Game">
  <img src="https://img.shields.io/badge/AI--assisted-Modding-purple.svg" alt="AI Assisted">
</p>

<p align="center">

[![Branch Sync Check](https://github.com/whozghiar/jak-project/actions/workflows/branch-sync-check.yaml/badge.svg?branch=jak3%2Ffeatures%2Fbattle_of_spargus)](https://github.com/whozghiar/jak-project/actions/workflows/branch-sync-check.yaml?query=branch%3Ajak3%2Ffeatures%2Fbattle_of_spargus)

</p>

> [!NOTE]
> The badge above is a native **GitHub Actions status badge** for
> [`branch-sync-check.yaml`](https://github.com/whozghiar/jak-project/actions/workflows/branch-sync-check.yaml),
> scoped to this branch — GitHub renders it live from that workflow's own run history,
> nothing generates or rewrites this image by hand. It goes green the moment this branch
> next merges `master-dev` cleanly (usually via the daily automated sync), and can turn
> red if someone pushes commits here without syncing first. It cannot turn red purely
> because `master-dev` moved on without a new push landing here — run `task modding-branch-status`
> or check GitHub Actions to audit fleet-wide mergeability.

> **Contents:** [Overview](#overview) · [Key Features](#key-features) · [Download & Play](#download--play-via-opengoal-launcher-players) · [Developer Setup](#developer-setup--local-compilation) · [Demo Video](#demonstration-video) · [Technical Documentation](#technical-documentation)

---

> [!NOTE]
> This mod moved from the `jak3/features/battle_of_spargus` branch of [whozghiar/jak-project](https://github.com/whozghiar/jak-project) to this repository. Earlier releases stay installable from the launcher catalog.

## Overview
Brief, simple description of what this mod introduces or modifies in the game.

- **Target Game:** Jak 3
- **Repository:** [`whozghiar/jak3-mod-battle-of-spargus`](https://github.com/whozghiar/jak3-mod-battle-of-spargus)

## Key Features
- **Feature 1:** Simple description of the first key feature.
- **Feature 2:** Simple description of the second key feature.
- **Feature 3:** Simple description of the third key feature.

## Download & Play via OpenGOAL Launcher (Players)

> [!TIP]
> **No developer environment required!** Players can install and play this mod directly using the official OpenGOAL Launcher:

### Option A — Add Custom Mod Source (Recommended)
1. In the **OpenGOAL Launcher**, navigate to **Settings ▸ Mods ▸ Add Custom Mod Source**.
2. Paste this catalog URL:
   ```text
   https://raw.githubusercontent.com/whozghiar/jak-project/jak3/features/battle_of_spargus/index.json
   ```
3. Go to the **Mods** tab, locate **Battle Of Spargus**, and click **Install**.
4. Select your clean PS2 game ISO when prompted. The launcher will automatically extract assets and launch the game!

### Option B — Manual Installation from GitHub Releases
1. Download the pre-built package for your operating system from the [Releases](https://github.com/whozghiar/jak-project/releases) tab (`windows-v*.zip` or `linux-v*.zip`).
2. Extract the archive into your OpenGOAL Launcher features directory:
   - **Windows:** `%APPDATA%\OpenGOAL-Launcher\features\jak3\mods\_local\battle_of_spargus\`
   - **Linux:** `~/.config/OpenGOAL-Launcher/features/jak3/mods/_local/battle_of_spargus/`
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
- **Status:** [Not required (GOAL-only mod, standard binaries sufficient) / `task build-release-game` (engine or compiler C++ changed) / `task build-release` + `task extract` (decompiler or decompiler/config changed)]
- **Details:** [Specify which C++ layer was modified — see `docs/modding/guides/build_and_iteration_workflow.md`]
```bash
# GOAL-only mod: nothing to build — go straight to the REPL below.
# Engine / compiler C++ changed:
task build-release-game
# Decompiler or decompiler/config changed (then step 3 is mandatory):
task build-release-decomp
```

### 3. Asset Extraction
- **Status:** [Required (`task extract`) / Standard extraction sufficient]
- **Details:** [Specify if custom 3D models, textures, or sound banks require extraction]
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

[![Demonstration Video](https://img.youtube.com/vi/YOUR_VIDEO_ID/maxresdefault.jpg)](https://youtu.be/YOUR_VIDEO_ID)

**[Watch the demonstration video on YouTube](https://youtu.be/YOUR_VIDEO_ID)**

> [!NOTE]
> *Demonstration videos must be hosted externally on YouTube to prevent repository bloating. Replace `YOUR_VIDEO_ID` with your YouTube video ID (e.g. `MnqnybexhSA` from `https://youtu.be/MnqnybexhSA`).*

## Technical Documentation
For the complete technical breakdown, architecture, and developer notes, refer to:
- [`docs/modding/current_mod/battle_of_spargus_readme.md`](docs/modding/current_mod/battle_of_spargus_readme.md)

---
*(AI-assisted)*
