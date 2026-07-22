# Modding

DRONECOM's chassis, sensors, warheads, and vehicle designs are all data — plain
RON (Rusty Object Notation) files, the same ones the shipped game loads. A mod
is a folder of those files plus a small manifest. Drop one into your mods
folder and the game merges its content in at startup; publish it to the Steam
Workshop and anyone can subscribe to it.

This guide covers the whole loop: what a mod folder looks like, how to write
and test one, and how to publish it. If you just want to see working examples,
jump straight to [`mods/examples/`](../mods/examples/) — five small mods this
guide walks through below.

## Contents

- [What a mod is](#what-a-mod-is)
- [The manifest: `mod.ron`](#the-manifest-modron)
- [The dev loop](#the-dev-loop)
- [Writing your first mod](#writing-your-first-mod)
- [Pricing](#pricing)
- [ID collisions](#id-collisions)
- [Using another mod's parts](#using-another-mods-parts)
- [Publishing](#publishing)
- [Multiplayer](#multiplayer)
- [Updating and removing mods](#updating-and-removing-mods)

## What a mod is

A mod is a folder that contains a manifest and some content files:

```
my-radar-pack/
├── mod.ron                      # required — name, author, description
├── preview.png                  # optional — Workshop thumbnail
├── workshop.ron                  # written by the game after your first publish
├── my_radar.component.ron       # a new sensor
└── my_radar_demo.blueprint.ron  # a drone that mounts it
```

- **`mod.ron`** is the only required file — it's how the game recognizes the
  folder as a mod at all. Its schema is documented in
  [`mod-manifest.md`](../data/docs/mod-manifest.md).
- **`preview.png`** is your Workshop item's thumbnail. If you don't include
  one, the game falls back to a default image on publish.
- **`workshop.ron`** records the Steam Workshop item id after your first
  publish, so a later publish updates the same item instead of creating a
  duplicate. The game writes this file for you — don't hand-edit it.
- **Content files** are named by what they define: `*.chassis.ron` for a new
  airframe/hull, `*.component.ron` for a new sensor or warhead, and
  `*.blueprint.ron` for a vehicle design. They must sit directly inside the
  mod folder — the game doesn't look in subfolders.

Only two of the four component categories can be modded right now:
**sensors** and **warheads**. Utility components (engines, fuel tanks, and
similar support gear) aren't moddable yet, nor is the carried-entities
category — a `.component.ron` declaring either is rejected with a diagnostic
explaining why. (Carrying another blueprint as a munition — a drone carrying
a missile, say — is unaffected; that's a property of the blueprint's
inventory, not a component you author.) Chassis have no such restriction: any
chassis form the base game supports can be modded.

A mod reaches the game one of two ways:

- **Local** — a folder you drop directly into your mods folder (dev
  iteration, or a mod someone sent you by hand).
- **Workshop** — a Steam Workshop item you've subscribed to. Steam downloads
  it and the game discovers it automatically; you never touch its files.

## The manifest: `mod.ron`

Every mod needs exactly one `mod.ron` at its root:

```ron
(
    name: "My Radar Pack",
    author: "Your Name",
    description: "A shorter-ranged, cheaper search radar.",
)
```

`name` and `description` seed your Workshop item's title and description the
first time you publish (you can still edit them on the Workshop page
afterwards — they aren't kept in sync). All three fields are required, and
an unrecognized field is rejected outright rather than silently ignored —
see [`mod-manifest.md`](../data/docs/mod-manifest.md) for the full field
reference.

## The dev loop

1. From the main menu, open **Mods**, then click **OPEN MODS FOLDER**. The
   game creates the folder if it doesn't exist yet and reveals it in your
   file manager.
2. Create a subfolder for your mod and add a `mod.ron` plus your content
   files.
3. Back in the Mods screen, click **RE-CHECK**. This re-scans every mod
   folder and reports fresh diagnostics — parse errors, validation failures,
   ID collisions — without touching the running game. Select your mod's row
   to see its full diagnostics or, if it loaded cleanly, the content it
   contributed.
4. Fix any errors shown and RE-CHECK again until the row reads **LOADED**.
5. **Restart the game to play with it.** Mods are merged into the game's
   registries once at startup — RE-CHECK only refreshes diagnostics, so a
   fixed or newly-added mod's content doesn't reach a purchase list or the
   designer until you restart.

## Writing your first mod

[`mods/examples/`](../mods/examples/) has five small, real mods that load
against the shipped game and are exercised in the developer's CI, so they
won't rot out from under this guide. Copy one as a starting point.

### The simplest mod: a vanilla blueprint

Start with
[`example-vanilla-blueprint`](../mods/examples/example-vanilla-blueprint/).
It's as small as a mod gets: a `mod.ron` and one `.blueprint.ron` that mounts
only shipped chassis and components. There's nothing to author from scratch
here beyond picking parts — it demonstrates that a blueprint built entirely
from base-game parts stays multiplayer-legal even while other mods are
loaded. See [`blueprints.md`](../data/docs/blueprints.md) for the full
blueprint schema.

### Adding a new part

[`example-chassis`](../mods/examples/example-chassis/),
[`example-sensor`](../mods/examples/example-sensor/), and
[`example-warhead`](../mods/examples/example-warhead/) each add exactly one
new part — a chassis, a sensor, and a warhead respectively — by copying a
shipped definition, giving it a new unique id, and tuning a couple of
numbers. Each is a complete, independent mod you could publish on its own.
Field references: [`chassis.md`](../data/docs/chassis.md) and
[`components.md`](../data/docs/components.md).

A mod that adds a new part and a demonstration blueprint together works the
same way, as long as both files live in the same mod folder — the loader
resolves a mod's own blueprints against its own definitions plus the base
game, so an intra-mod reference like this always works.

### Combining parts from several mods

[`example-blueprint`](../mods/examples/example-blueprint/) is a `mod.ron`
plus a single `.blueprint.ron` that mounts the chassis, sensor, and warhead
from the three mods above — none of which live in its own folder. This works
at load time because the game unions every *installed* mod's definitions
before validating blueprints, so a cross-mod reference resolves as long as
the mod that defines the part is also installed. Drop all five example mods
into your mods folder together (not just `example-blueprint` on its own) to
see this resolve.

This shape — a manifest plus nothing but a blueprint file, referencing parts
that live in other mods — is also exactly what a design published from the
in-game Drone Designer looks like once it reaches disk. See
[Using another mod's parts](#using-another-mods-parts) below for what that
means for publishing.

## Pricing

Don't author a cost in your `.chassis.ron` or `.component.ron` — there's no
`cost` field to set. The game prices every part from its capability stats
using the same pricing formula it uses for shipped content, at load time.
Change a stat and the price follows; there's no way to author an arbitrary
price directly.

## ID collisions

Every chassis, component, and blueprint id must be unique — but what happens
next depends on what it collides with:

- **A shipped id, or another installed mod's id**: caught up front, before
  anything merges. The whole mod is rejected — none of its content loads —
  and RE-CHECK names exactly which id collided and with what.
- **One of your own local designs**: this can only be caught once a mod's
  content actually merges into the running game at startup — RE-CHECK can't
  see your local designs, so it won't catch this case. Only the individual
  conflicting blueprint is skipped; the rest of the mod (its chassis,
  components, and any other blueprints) still merges normally. The mod's row
  shows an error naming the collision, so you'll notice it even though most
  of its content loaded.

Either way there's no override: rename one of the colliding ids, then
RE-CHECK (for a base-game or mod-vs-mod collision) or restart (for a
local-design collision, since only a restart re-runs the merge) to confirm
it's resolved.

## Using another mod's parts

There are two different ways to reference content from another mod, and they
have different rules at publish time:

- **A mod folder that references another mod's parts** (like
  `example-blueprint` above) needs that other mod to also be installed for
  its blueprints to resolve — both locally, and for anyone who subscribes to
  it on the Workshop. When you publish a mod folder from the **Mods** screen
  (see below), the game re-validates that folder in isolation against the
  base game right before uploading, so this only works cleanly when every
  part a mod's blueprints need is either shipped or bundled in the same
  folder.
- **A vehicle design saved in the Drone Designer** can freely use parts from
  any mod you currently have installed — the designer validates against your
  live session, which includes every loaded mod. Publishing a design (via the
  Designer's upload icon — see below) does **not** re-validate it against
  modded parts, so a design that depends on another mod's content publishes
  without complaint.

Either way, the dependency isn't automatically declared to Steam. If your
mod or design needs another mod's content, tell your subscribers explicitly:
set **Required Items** on your Workshop item's page (edit it after
publishing) to point at the mod(s) it depends on. Steam then makes sure
subscribing to yours pulls theirs in too. Skip this and a subscriber without
the dependency will see a missing-reference error instead of your content.

One current limitation: publishing a design that nests another one of your
own designs (for example, a drone carrying a custom missile you also
designed) isn't supported yet — only the top-level design is included, so a
subscriber sees a missing-reference error for the nested one. Keep a
published design to a single, self-contained blueprint for now.

## Publishing

Publishing needs Steam running and connected — every publish affordance is
disabled with an explanation when it isn't.

### Publishing a mod folder (parts)

On the **Mods** screen, each of your local mods has an **UPLOAD** button
(or **UPDATE**, once you've published it before). It's disabled if Steam is
unavailable, if the mod currently fails to load (a broken mod is never
uploaded), or if another publish is already running. Clicking it re-validates
the folder one more time, then uploads its content — including any
`.blueprint.ron` files it contains — as one Workshop item, tagged
automatically by what it contributes (chassis, sensors, warheads, and/or
blueprints). If the folder has a `preview.png`, that becomes the item's
thumbnail.

### Publishing a design

In the **Drone Designer**'s library, each of your own saved designs (not a
default or an already-published Workshop design, which are read-only) has an
upload icon — tooltip **Publish to the Workshop**. It's disabled if Steam is
unavailable, another publish is running, or the design itself doesn't
validate (weight, category limits, and so on) — but a design that uses
modded parts, or nests another design, is otherwise publishable; see
[Using another mod's parts](#using-another-mods-parts) above for what that
means in practice. Publishing stages your design as a small one-blueprint mod
behind the scenes: the item's title is your design's codename, the author is
your Steam persona name, and the description is your design's designator and
chassis name.

### First publish: accept the legal agreement

Once an upload finishes, the game opens your item's Workshop page in your
browser. **The first time you publish anything, Steam only makes the item
visible to others after you accept the Workshop Legal Agreement on that
page** — so don't skip that step. You can always get back to an item's page
later from its **VIEW PAGE** link on the Mods screen.

Published items are public by default. Use the Workshop page itself if you
want to change an item's visibility, tags, or Required Items after
publishing.

## Multiplayer

A vehicle design is single-player-only if it uses **any** modded chassis or
component, from any mod — a **MOD PARTS** badge marks it wherever it's
listed (the purchase catalog, your design library, and the fleet picker), so
you can tell before you're in a lobby. A design built entirely from
shipped parts stays multiplayer-legal even while other mods are loaded (that
badge is what `example-vanilla-blueprint` demonstrates). Trying to bring a
modded design into a network session — the lobby's fleet check, the host's own
starting fleet, or a live purchase — is refused, with a tooltip naming the
offending parts in the UI-facing cases.

Definition mods (new chassis, sensors, warheads) only affect designs that
actually use them — installing one doesn't make your whole fleet
single-player-only, only the specific designs built from its parts.

## Updating and removing mods

RE-CHECK never removes a mod's content from the running game — restart to
pick up an update, exactly as when installing one for the first time.

Removing or unsubscribing from a mod can invalidate single-player saves that
used its content — a save that references a design, chassis, or component
the mod provided has nothing to load in its place once the mod is gone. Keep
a mod installed for as long as you want to keep loading saves that depend on
it.
