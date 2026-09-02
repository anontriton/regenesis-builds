<h1 align="center">Re: Genesis</h1>

<p align="center">
  <em>A grid-based tactical RPG about a losing rebellion — and the people still fighting it.</em>
</p>

<p align="center">
  <img alt="Engine: Godot 4.4" src="https://img.shields.io/badge/engine-Godot%204.4-478cbf?logo=godotengine&logoColor=white">
  <img alt="Language: GDScript" src="https://img.shields.io/badge/language-GDScript-355570">
  <img alt="Platforms: Windows | macOS" src="https://img.shields.io/badge/platforms-Windows%20%7C%20macOS-lightgrey">
  <img alt="Version: 0.0.29" src="https://img.shields.io/badge/build-0.0.29-informational">
</p>

<p align="center">
  <a href="https://github.com/anontriton/regenesis-builds/releases"><strong>⬇ Download the latest build</strong></a>
</p>

---

<p align="center">
  <img src="docs/screenshot-battle.png" alt="A battle in Re: Genesis — Thomas is selected, with blue tiles showing his movement range and red tiles showing the enemy threat range layered on top." width="100%">
</p>

<p align="center"><sub>Selecting a unit projects its full threat envelope: blue for reachable tiles, red for everything an enemy can hit from where <em>they</em> can move.</sub></p>

---

## About

**Re: Genesis** is a turn-based tactics game in the tradition of *Fire Emblem* and *Final Fantasy Tactics*, built solo in Godot 4 as my master's capstone project at Tufts University.

Two years into the Genesis Rebellion, the Rathos Empire has not broken. You play Thomas, one of the rebellion's commanders, sent downriver to the neutral settlement of Yore to beg for steel and grain before an Imperial legion arrives to take both. It does not go well.

The build here is a **vertical slice**: a full playable chapter that runs the complete game loop — visual-novel dialogue, a scripted tactical battle with story triggers, post-battle resolution — on top of systems built to carry a much longer campaign.

## Download & play

Builds are published on the [**Releases page**](https://github.com/anontriton/regenesis-builds/releases). Grab the file for your platform, and see the notes below.

| Platform | File | Notes |
| --- | --- | --- |
| Windows 10/11 | `ReGenesis Windows <version>.exe` | Run it directly. SmartScreen may warn — choose *More info → Run anyway*. |
| macOS 11+ (Apple Silicon & Intel) | `ReGenesis MacOS <version>.dmg` | Universal binary, unsigned; see the unblock step below. |

**System requirements** are light — the game targets Godot's OpenGL compatibility renderer, so integrated graphics are fine. It runs at 1920×1080 fullscreen. A mouse is strongly recommended.

<details>
<summary><strong>macOS: "Re: Genesis is damaged and can't be opened"</strong></summary>

<br>

These builds aren't signed with an Apple Developer certificate, so macOS quarantines them on download. To clear the quarantine flag, drag the app to `/Applications`, then run:

```bash
xattr -dr com.apple.quarantine "/Applications/Re: Genesis.app"
```

Then open it normally. (If you'd rather not run that, right-click the app → *Open* → *Open* also works on some macOS versions.)

</details>

## Controls

The game is mouse-first, with keyboard shortcuts and partial controller support.

| Input | Action |
| --- | --- |
| **Left click** | Select a unit / confirm a destination or target |
| **Left click again** | Open the selected unit's action menu |
| **Right click + drag** | Pan the camera |
| **Mouse wheel** | Zoom in / out |
| **1 – 4** | Command · Ability · Item · Wait |
| **Enter** | Confirm a combat preview · advance dialogue |
| **Esc** | Cancel out of targeting, or open the battle menu |
| **Tab** | Muster Roll — the full party roster and status screens |

A unit can **move and attack in the same action**. Click an empty tile in range to move; click an enemy in range to attack; click an enemy that's only reachable by moving first, and you'll get a combat forecast before committing.

## What's in the game

**Tactics layer**
- Grid movement with terrain-cost pathfinding and per-unit terrain affinities — a mage crossing sand isn't paying the same price a knight is
- A combined threat overlay that computes what enemies can hit *after moving*, not just from where they stand
- Combat forecasts before every attack: damage, hit chance, crit chance, initiative order, and lethal warnings
- Counterattacks, glancing blows against high-defense targets, and positional bonuses for surrounding a target

**Combat math**
- A physical weapon triangle (Blunt / Slash / Pierce) layered against three armor classes
- A six-element cycle — Fire › Ice › Wind › Earth › Lightning › Water › Fire — at 1.5× effective, 0.5× resisted
- Physical/special attack and defense split, with damage variance, luck-scaled crit, and speed-scaled accuracy

**Enemy AI**
- Utility-scored decision making that weighs expected damage, target health, accumulated aggro, positional safety, and proximity to allies
- Situational nerve: enemies press the attack when they outnumber you locally, hesitate when isolated and hurt, and healers triage allies with emergency priority below 15% HP
- A small randomized term so the same board doesn't replay identically

**Campaign & content**
- 7 playable protagonists, 7 enemy archetypes, 20 abilities, 12 weapons — all defined in JSON and hot-swappable without touching engine code
- Visual-novel dialogue scenes with portraits, backgrounds, and history scrollback
- A level-scripting event system: trigger popups, dialogue, camera moves, reinforcements, and objective changes on turn counts, HP thresholds, unit deaths, or position entry
- Six mission objective types, persistent party progression between battles, and a shared inventory convoy

## Under the hood

For anyone curious about the engineering rather than the game — roughly **15,000 lines of GDScript across ~48 files**, written solo over the course of the capstone.

The architecture splits into two orchestration layers:

- **A macro layer** (`GameRoot`) owns the long-lived objects — the data registry, session state, and campaign progression — and swaps the active scene through a state machine: `TITLE → DIALOGUE → BATTLE → POST_BATTLE → ENDING`. Scenes receive their dependencies by injection rather than reaching for globals, which keeps any scene runnable standalone in the editor.
- **A battle layer** (`BattleScene`) owns the short-lived tactical managers — turn flow, objectives, combat resolution, AI, and level events — and coordinates them entirely through Godot signals rather than direct calls between siblings.

Two design decisions did most of the heavy lifting:

**A serialized UI queue.** Tactics games generate a lot of things that all want the screen at once — a turn-change banner, a combat animation, a scripted story popup, a unit dying, a camera pan to the reinforcements that just arrived. Rather than letting those race, the battle state machine has a dedicated `SHOWING_UI` state ("cinema mode") that drains a queue in order while blocking input, then returns to whatever state it interrupted. Death checks and objective evaluation are deliberately deferred until the board settles, so a popup can't cut a combat animation in half.

**Data-driven everything.** Campaign structure, level layouts, unit archetypes, abilities, weapons, items, and dialogue all live in JSON and plain-text files parsed at load. Adding a chapter, rebalancing a weapon, or scripting a new mid-battle story beat is a data change, not a code change — which is what made solo iteration on balance feasible at all.

The project also ships an **automated in-engine test suite** (~20 test groups) that runs against a live battle scene in debug builds, covering the damage pipeline's bounds and invariants, type effectiveness, ability targeting rules, turn-state flags, aggro decay, buff expiry, and UI-sequencing integration cases like "does the queue actually drain and hand control back to the player."

## Where it stands

Version **0.0.29** is the capstone presentation build — a complete, playable vertical slice rather than a finished campaign. One chapter, built to prove the systems underneath it.

Current limitations, stated plainly:

- The campaign is one chapter; the content pipeline supports many more
- Saving isn't implemented yet — the menu option is deliberately disabled rather than hidden
- Three of the six objective types (`PROTECT_UNIT`, `REACH_LOCATION`, `CUSTOM`) format their objective text correctly but their win-checks are still stubs
- Combat previews are best-effort — displayed crit chance omits a few context-dependent bonuses that the real resolution does apply
- Buff effects stack where they should refresh
- The post-battle screen reports the result but not per-unit battle stats

## Roadmap

**Next up — finishing the loop**
- **Save/load.** The snapshot system that powers mid-battle retries already serializes party state; save files are largely a matter of persisting that to disk and restoring campaign position on load.
- **The remaining objective types.** Escort and seize missions are the two that most change how a map plays, and they're the ones still stubbed.
- **Combat animation.** There's a reserved animation window in the battle UI sized for Fire Emblem-style attack cutaways; right now it holds the lighter sprite-slide feedback that shipped instead.
- **Post-battle results.** Per-unit damage dealt, kills, and level-ups on the victory screen.

**Then — content**
- **More chapters.** The campaign format takes arbitrary chapter counts today; the bottleneck is authored content — maps, story text, and encounter balance — not engine work.
- **A cutscene system.** The event scripter already has a `trigger_cutscene` hook wired and waiting behind a stub.
- **A deeper item and inventory layer.** Two consumables ship today; the inventory convoy and effect system are built to carry equipment swaps and a wider consumable set.

**Longer term**
- Multi-faction battles — teams became named strings rather than hardcoded 0/1 early on specifically to allow third parties on the field
- Permadeath as a difficulty option, and the support-conversation systems the genre is known for

## Source code

This repository hosts **playable builds only**. The game's source lives in a private repository — I'm happy to walk through the code, the architecture, or any specific system with employers and collaborators; just reach out.

## Credits

Designed, programmed, and written by **Iverson Lai**.

Built with [Godot Engine](https://godotengine.org/) 4.4.

---

<p align="center"><sub>© Iverson Lai. All rights reserved. Builds are provided for evaluation and play; the game's code and assets are not licensed for redistribution or reuse.</sub></p>
