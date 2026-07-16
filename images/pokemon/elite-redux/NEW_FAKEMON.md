<!--
SPDX-FileCopyrightText: 2026 Pagefault Games
SPDX-License-Identifier: AGPL-3.0-only
-->

# New ER fakemon / mega / primal sprite art (2026-07)

Battle sprite art for 16 new ER custom species/forms, added under the existing
`images/pokemon/elite-redux/<slug>/` convention (TexturePacker `front.png`/`.json`
… pairs; `front`/`back`/`shiny`/`shiny-back` are 64×64, `icon` is 32×64 with a
32×32 frame; atlas `meta.app = er-build/generate-er-sprite-atlases`).

## Slugs added (game key → er-assets folder)

| game key (slug prefix in code) | er-assets slug / folder | files |
|---|---|---|
| er_astoot            | `astoot`             | front, back, icon |
| er_discupid          | `discupid`           | front, back, icon, shiny, shiny-back |
| er_mega_electivire_x | `electivire_mega_x`  | front, back, icon, shiny-back |
| er_mega_dragonite_z  | `dragonite_mega_z`   | front, back, icon |
| er_mega_hydreigon_x  | `hydreigon_mega_x`   | front, back, icon, shiny, shiny-back |
| er_mega_jumpluff     | `jumpluff_mega`      | front (anim), back (anim), icon |
| er_mega_minun        | `minun_mega`         | front, back, icon, shiny, shiny-back |
| er_mega_plusle       | `plusle_mega`        | front, back, icon, shiny, shiny-back |
| er_mega_parasect     | `parasect_mega`      | front, back, icon |
| er_mega_skarmory_z   | `skarmory_mega_z`    | front, back, icon |
| er_mega_shuckle_y    | `shuckle_mega_y`     | front, back, icon, shiny, shiny-back |
| er_mega_xerneas      | `xerneas_mega`       | front, back, icon, shiny, shiny-back, shiny-2, shiny-3, shiny-back-2, shiny-back-3 |
| er_primal_mew        | `mew_primal`         | front, back, icon, shiny, shiny-back |
| er_primal_regigigas  | `regigigas_primal`   | front, back, icon, shiny, shiny-back |
| er_tentalect         | `tentalect`          | front (anim), back, icon |
| er_regitube          | `regitube`           | front (anim), back (anim), icon, shiny (anim), shiny-back (anim) |

The redirect resolves `elite-redux/{slug}/{front,back,shiny,shiny-back,icon}` —
see `src/data/elite-redux/er-form-sprite-redirect.ts` / `ErCustomSpecies` in the
game repo, and the `er-sprite-manifest.ts` entries the game side must add.

## Animated sprites (jumpluff, tentalect, regitube)

There is no pre-existing *animated fakemon* precedent in this repo — every other
`elite-redux/<slug>/` is single-frame. The game builds a sprite's animation with
Phaser `generateFrameNames({ zeroPad: 4, suffix: ".png", start: 1, end: 400 })`
at `frameRate: 10, repeat: -1` over the loaded `front`/`back` atlas. So the
animated sprites here are ordinary TexturePacker atlases whose frames are named
`0001.png … 000N.png` (a vertical 64×64 strip). Frame counts are resampled to
**10 fps** (100 ms/frame) so fixed-rate playback matches the source loop length:

- `jumpluff_mega/front` 44 frames, `/back` 39 frames (from the two source GIFs).
- `tentalect/front` 44 frames (downscaled **0.164×** from the 480×480 GIF,
  nearest-neighbour, to big-mon scale). `back` is a static 64×64 frame.
- `regitube/{front,back,shiny,shiny-back}` 24 frames each (baked `idle20`
  cell-animation bank; 116.7 ms → 10 fps).

A single-frame reader (`0001.png`) still gets a valid full sprite, so these
degrade gracefully anywhere the anim isn't built.

## Notes / caveats for the game-side wiring step

- **regitube icon is a PLACEHOLDER.** `regitube/icon.png` is a downscaled battle
  frame (the ROM ships no box icon for #377). Real hand-drawn 32×32 icon art is
  still wanted; a downscaled 96px battle sprite does not read cleanly at 32px.
- **electivire_mega_x shiny FRONT omitted.** The provided shiny-front source was a
  corrupt 32×32 gradient blob (unusable). The valid shiny **back** is included;
  the shiny front falls back to the game's palette-shift of the normal front.
- **astoot has no shiny art** — intentional; uses the standard palette-shift shiny
  fallback (no `shiny*.png` files).
- **tentalect cry** is at `audio/cry/tentalect.wav`. The cry-dir convention is
  `.m4a`, but no AAC encoder was available when this art was prepared, and ER
  custom species currently load sprite-only (no cry). Transcode to
  `audio/cry/tentalect.m4a` if/when a cry is wired.
- **Derived icons.** `regigigas_primal`, `shuckle_mega_y`, `mew_primal`,
  `xerneas_mega`, `hydreigon_mega_x`, `minun_mega`, `plusle_mega` had no dedicated
  icon source; their `icon.png` is downscaled from the front sprite.
- **xerneas_mega** ships 4 palette tiers (normal + shiny + shiny-2 + shiny-3, with
  matching backs), sliced from the 4 palette columns of the source sheet; the
  leftmost (dark-body) column is treated as normal.
- **Partner Eevee: NO art exists yet.** No partner-Eevee sprite was provided and
  none was invented. The game side uses placeholder redirects for it.

## Slice corrections (2026-07 re-audit)

- **hydreigon_mega_x RE-SLICED (bug fix).** The first pass mis-sliced every
  `hydreigon_mega_x` sprite: `front`/`shiny` were the *head-cluster only* (the three
  heads with no body/tail — this read in-dex as "three small sub-sprites side by
  side"), and `back`/`shiny-back` were a fragment (one arm-head + tail loop), not a
  real render. Re-derived from `Mega_hydreigon_x_sheet.webp` (600×398, transparent
  bg with opaque-black filler blocks on the left). **True layout:** top half =
  normal palette, bottom half = shiny (purple manes / orange heads). Each half is a
  left-to-right size ladder — small icon/anim frames on a black block (x 0–200),
  medium front+back (x 220–310), large front+back (x 323–413), and a single **big
  showcase front** filling the right column (x 426–594, the full half-height).
  Slices used: `front` = normal big showcase (comp bbox 426,13–594,181);
  `shiny` = shiny big showcase (426,211–594,379); `back` = normal large back
  (334,110–413,188); `shiny-back` = shiny large back (334,308–413,386); `icon`
  down-scaled from the front. Extraction is by connected-component (each sprite is
  an isolated alpha blob), so no neighbour bleed and the near-black body is never
  eaten. All five outputs Read-verified as one complete Pokémon each.
- **minun_mega / plusle_mega verified CORRECT (no art change).** Re-checked the
  shared `Mega_minun_and_plusle_front_back_icon_sheet.webp` (905×237). Published
  slices are complete, single-sprite, transparent, with the right per-variant
  palette (Minun front = blue / shiny = green; Plusle front = red / shiny = orange),
  matching the source small-frame grid. The sheet has a **big normal-colour
  showcase** for each (Plusle x 273–428, Minun x 717–870) but **no shiny showcase**,
  so front and shiny must both come from the small frames to keep normal/shiny at a
  matching scale — which is exactly what shipped. The reported "no image in the
  Pokédex" for these two is therefore **not an asset defect** (PNGs are non-empty
  and the atlas JSON is valid) — it points to a game-side dex loader / form
  registration issue for `er_mega_minun` / `er_mega_plusle`; flagged for the
  game-side agent.
- **xerneas_mega verified CORRECT (no art change).** All 8 variant sprites are clean
  single Xerneas. The `shiny` (Active-mode black body) and `shiny-3` (silver/teal)
  fronts *look* busy/"doubled" at 64px but are single animals — confirmed by
  matching them pixel-for-pixel against clean cell re-slices of the source.
