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
- [Optional chassis artwork](#optional-chassis-artwork)
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
  one, the game generates one from your mod's content on publish (see
  [Publishing a mod folder](#publishing-a-mod-folder-parts)).
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

`name` and `description` become your Workshop item's title and description,
and the game appends a stats section for each chassis and design in the mod.
Every publish sets them again, so make lasting edits in `mod.ron` rather than
on the Workshop page. All three fields are required, and
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

### Optional chassis artwork

Chassis illustrations are optional. Put an SVG beside its chassis definition
with the same stem: `my_airframe.chassis.ron` uses
`my_airframe.svg`. The game applies this rule identically to built-in and
modded chassis. It considers artwork only for the definition source that was
accepted into the registry, so a rejected mod cannot supply artwork for an
otherwise accepted chassis.

No SVG is required. An absent sidecar is normal. An unreadable, invalid, or
unsupported sidecar produces a non-blocking warning; the chassis still loads.
In either case the UI shows no artwork, placeholder, substitute symbol, or
empty image space. Restart after adding or changing a sidecar: artwork is
loaded with the chassis definitions at startup.

#### SVG outline subset

Author a compact, outline-only vector drawing. The SVG root may set a `viewBox`
(and optional dimensions, `preserveAspectRatio`, or version); use `g` groups
and `path` elements for the drawing, with optional `title` and `desc`. Paths
may use normal path geometry and affine transforms. Every visible path needs
an opaque solid stroke and a positive stroke width; use `fill="none"`.
Supported caps are `butt`, `round`, and `square`; supported joins are `miter`,
`miter-clip`, `round`, and `bevel`.
Presentation properties may be attributes or simple `name: value;` inline
declarations. CSS comments and `!important` are outside this subset. Path data
and canvas attributes must parse completely; malformed trailing data is rejected.

The root may also declare `data-nose="up"` or `data-nose="right"`: the
direction the drawn nose points. The designer then draws the outline to scale
on a metre grid, treating the drawing's extent along that axis (path
centrelines, nose to tail) as the chassis `length`. Draw top-down aircraft
nose-up or nose-right and side-on hulls, helicopters and munitions nose-right.
Without `data-nose` the outline still renders, on a plain grid with no scale
caption; any other value rejects the sidecar.

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 120" data-nose="right"
     fill="none" stroke="currentColor" stroke-width="1.6"
     stroke-linecap="round" stroke-linejoin="round">
  <title>My airframe</title>
  <path d="M12 60 H308 M80 60 L140 20 M180 20 L240 60"/>
</svg>
```

The UI applies its own semantic tint, so artwork should use `currentColor`
rather than encode a gameplay meaning in its stroke color. Keep the `viewBox`
tight around the art: previews preserve its aspect ratio.

[`example-skimmer`](../mods/examples/example-skimmer/) ships a complete
worked example: `example_sk_90.svg` beside `example_sk_90.chassis.ron`.

Do not use fills, transparency, dashed strokes, non-default vector effects,
filters, masks, clipping, gradients or other URL paint, animation, text,
images, external references, DTDs, processing instructions, or other SVG elements.
One unsupported feature rejects the entire sidecar rather than partially
rendering it.

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

Publishing a design that nests another one of your own designs (for example,
a drone carrying a custom missile you also designed) bundles the whole
family: every one of your own designs it carries, however deeply nested, is
staged into the same upload, so a subscriber's install resolves them without
needing your local library. A shipped default nested the same way needs
nothing extra — subscribers already have it — and a design nested from
another mod is an external dependency like any other: see Required Items
above, since it is never copied into your upload.

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
blueprints). A mod with only a `mod.ron` has nothing to publish, so its button
is disabled.

The item's description is your `mod.ron` description followed by one section
per chassis and per design — headed **Chassis:** or **Blueprint:** and its
name — listing the same stats the Drone Designer shows (in your language and
unit settings). A design's section also lists, within it, what that design
carries under the designer's own headings: COMPONENTS, LOADOUT, and STORES. Sections that would
push the description past Steam's limit are left off.

If the folder has a `preview.png`, that becomes the item's thumbnail.
Otherwise the game generates one from the first of these it finds:

1. A chassis with [artwork](#optional-chassis-artwork): its outline centred
   on the designer's grid panel, which fills the image.
2. A design whose chassis has artwork (yours or a built-in chassis): its
   chassis outline, the same way.
3. A chassis or design without artwork: its classification icon.
4. A sensor, warhead, or decoy: the Drone Designer's icon for its category,
   in the same color.

Generated thumbnails are 512×512 squares with no text: Steam shows the
item's title beside them.

### Publishing a design

In the **Drone Designer**'s library, each of your own saved designs (not a
default or a subscribed Workshop design: those are read-only, and editing one
saves your changes as a new design of your own) has an
upload icon — tooltip **Publish to the Workshop**. It's disabled if Steam is
unavailable, another publish is running, or the design itself doesn't
validate (weight, category limits, and so on) — but a design that uses
modded parts, or nests another design, is otherwise publishable; see
[Using another mod's parts](#using-another-mods-parts) above for what that
means in practice. Publishing stages your design — plus every one of your
own designs it nests, however deeply — as a small mod behind the scenes: the
item's title is your design's codename, the author is your Steam persona
name, and the description is your design's designator and chassis name,
followed by the design's stats section.

The Workshop thumbnail is generated for you from the design itself — its
chassis outline on the designer's grid panel when the chassis has artwork,
otherwise its classification icon (in your designer's per-class color) — so
it always matches what's actually in the upload.
Unlike a mod folder (above), a design has no `preview.png` of its own to
override this with.

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

## Enabling and disabling mods

Each row on the Mods screen has an **ENABLED** checkbox. Untick it to stop a
mod loading without unsubscribing or deleting its folder; tick it again to
bring the mod back. The choice is saved in `settings.ron` and survives
restarts, and a mod you remove and later reinstall or re-subscribe to keeps
it.

Like installing a mod, the change applies **the next time you start the
game**. Until then the row shows **RESTART TO APPLY** and a warning heads the
Mods screen. The rows update at once to show what the next launch will load:
a disabled mod reads **DISABLED**, and any mod built on its content shows
the missing reference it will hit.

To the game a disabled mod is the same as an uninstalled one. Designs built
on its parts are reported as unavailable in the designer, and saved units
built from them are dropped from a save as described below. A disabled local
mod can't be published until you enable it again.

Removing or unsubscribing from a mod does not stop a single-player save from
loading. Units built from that mod's chassis or components are dropped from
the save; everything else — groups, deck queues, bookmarks and the contact
picture — loads without them. Keep a mod installed for as long as you want to
keep those units.
