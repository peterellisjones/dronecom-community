# DRONECOM — Community

This is the community hub for **[DRONECOM](https://dronecomgame.com)**, a naval drone-warfare strategy game currently in private playtesting. The game's source lives in a private repository — this repo is where players report bugs, share feedback, ground balance discussion in the real game data, and contribute translations.

## Reporting a bug

Open a [bug report](../../issues/new/choose). The form asks for your game version and OS — screenshots, save files, and logs can be dragged straight into any text box.

Before filing, a quick [search of existing issues](../../issues?q=is%3Aissue) helps avoid duplicates. If your bug is already reported, add a 👍 reaction — it helps prioritise.

## Feedback and ideas

- **Concrete suggestions** (balance, UX, missing features) → the [feedback form](../../issues/new/choose).
- **Open-ended impressions, questions, chat** → [Discussions](../../discussions) or the [Discord](https://discord.gg/MMzQCeQPfG).

## Game data (`data/`)

A read-only mirror of the game's data files, updated automatically on every release. These are the same files that ship inside the game's install folder — the mirror only ever shows shipped state, and its commit history is an exact per-release balance changelog (diff two mirror commits to see what changed between builds).

Use it to ground balance discussion: link the exact file and line in an issue. Please don't open pull requests against `data/` — it is overwritten by the next release's mirror. Balance proposals go in an [issue](../../issues/new/choose) with the `balance` label instead.

### File index

| Path | What it is |
|---|---|
| `data/blueprints/` | The stock vehicles: each `*.blueprint.ron` is a chassis plus its component loadout (carrier, drones, missiles, torpedoes, submarines, USVs). |
| `data/definitions/chassis/` | Chassis definitions (`*.chassis.ron`): airframes and hulls with their physics, handling, and capacity parameters. |
| `data/definitions/components/sensors/` | Sensor component definitions: radars, IR/optical sensors, ESM/warning receivers, and sonars. |
| `data/definitions/components/warheads/` | Warhead component definitions. |
| `data/config/ai.ron` | AI opponent pacing: evaluation, launch, and procurement intervals. |
| `data/config/ai-strategy.ron` | AI strategy-layer tuning: objective commitment windows and posture weighting. |
| `data/config/pricing.ron` | The capability-pricing model — every part's cost is derived from these parameters. |
| `data/config/simulation.ron` | Core simulation tuning: autopilot, deck operations, sensors, tasking. |
| `data/config/spatial_awareness.ron` | AI spatial threat weighting and coverage decay. |
| `data/config/trajectory.ron` | Munition trajectory shaping parameters. |
| `locales/` | All UI text — see Translations below. |

## Translations (`locales/`)

The game's UI text lives in `locales/*.yaml`, one file per UI module. Each key carries all languages inline (English is the source language; the game ships 11 locales, and CJK text has its own font fallbacks in-game).

Translation fixes and improvements are **welcome as pull requests**. The mirror itself is one-way: an accepted change is applied to the game's private repository by the developer, ships in the next release, and then reappears here in that release's mirror commit — so your PR being merged into the game can slightly lag the PR itself.

## Modding

Modding support is planned, with mods published via Steam Workshop. This repo may host modding documentation and example mods; exactly what lives here is still to be decided.

## Links

- Website: <https://dronecomgame.com>
- Discord: <https://discord.gg/MMzQCeQPfG>
