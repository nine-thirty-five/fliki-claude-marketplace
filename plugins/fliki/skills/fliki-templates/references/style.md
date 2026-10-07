# Style — stored units, aspect ratios, fonts, palettes

Almost nothing is stored in pixels. Design in px on the canvas, then convert with `template_units`;
every value below has an exact conversion and is invisible when guessed wrong.

## Stored units

| field | stored as | `template_units` value |
|---|---|---|
| `left`, `width` | % of canvas **width** | `left`, `width` |
| `top`, `height` | % of canvas **height** | `top`, `height` |
| `font.size` | 1–100 on an exponential curve | `fontSize` (px) |
| `font.lineHeight` | −100..100; `0` = 1.2em (omit it) | `lineHeight` (ratio, e.g. 1.05) |
| `font.letterSpacing` | −20..100, relative to the type size | `letterSpacing` (px) + `fontSize` in the same call |
| `font.padding` | 0–100 of canvas width; vertical is 0.6× horizontal | `padding` (horizontal px) |
| `radius` | 0–100 of half the box's shorter side; `100` = circle or pill | `radius` + `box` |
| `stroke.width`, `border.width` | 0–100; 100 = 6% of the canvas's shorter side | `stroke` |
| `blur` | 0–100 → 0–20px | `blur` |
| `opacity` | 0–100 (not 0–1) | — |
| `angle` | degrees | — |
| `shadow`, `textShadow` | raw px `{ top, left, blur, spread?, color }` (`textShadow` has no spread) | — |
| `filters` | `brightness contrast exposure saturation hue` −100..100 (0 = none); `sharpen noise vignette` 0–100 | — |

- Geometry is always % of the **canvas**, also for layers inside a container. Absent box = full
  bleed (`left 0, top 0, width 100, height 100`) — omit all four for a full-bleed layer.
- `radius` may be `[topLeft, topRight, bottomRight, bottomLeft]`.
- `fill`: hex (`#1B2433`, `#1B243380` with alpha), `transparent`, or a CSS gradient string —
  `linear-gradient(to bottom, #1B243300 0%, #1B2433E6 100%)` is a scrim with no css.
- `fit: { mode: cover|contain|fill, position? }`; `crop`: see layers. `flipX`/`flipY` booleans;
  `overflow: visible|hidden`; `blend`: a CSS blend mode.
- Never in `style`: `x`, `y`, `z`, `scale`, `scaleX`, `scaleY`, `rotation`, `rotate`, `skewX`,
  `skewY`, `translateX`, `translateY`, `transform` — use `angle`, `flipX`/`flipY`, `width`/`height`,
  and tweens for motion. Never `height: -1` or `style.layout`.
- Time is seconds everywhere.

`font.size` is resolution-independent: the same number renders larger on a taller canvas. For
sanity checks only (still convert with the tool):

| stored | 10 | 20 | 30 | 40 | 50 | 60 | 70 | 80 | 90 |
|---|---|---|---|---|---|---|---|---|---|
| px, 1080 tall (16:9, 1:1) | 25 | 36 | 46 | 57 | 68 | 83 | 106 | 150 | 243 |
| px, 1920 tall (9:16) | 45 | 64 | 82 | 101 | 122 | 147 | 188 | 267 | 431 |

The reachable range is about 16–432px at 1080 tall and 28–768px at 1920 tall.
`template_units { canvas, fontTable: true }` returns the full table.

## Aspect ratios (`sizeProps`)

| ratio | canvas |
|---|---|
| `16:9` | 1920 × 1080 |
| `9:16` | 1080 × 1920 |
| `1:1` | 1080 × 1080 |

Base values are for `settings.aspectRatio`. Every other declared ratio is a sparse patch on the
node (scene, layer or `subtitle`) that differs:

```json
"sizeProps": { "9:16": { "style.left": 7.407, "style.top": 56.771, "style.width": 85.185,
                         "style.height": 29.167, "style.font.size": 37 } }
```

- Keys are literal dotted paths under `data`, `style`, `timing`, `css`, `gsap`, `isHidden`, or
  (scenes) `customDuration`. Never `meta`, never the base ratio's key.
- Patch leaves. `"style.font": { … }` replaces the whole font object and drops every field you
  did not restate (`sizeProps.branchValue`).
- `null` removes a value at that ratio (`"style.crop": null`). `"isHidden": true` drops a layer
  there — often right for decoration with nowhere to go.
- A per-ratio `gsap` replaces the whole program; normally leave motion alone (`cw`/`ch` travel).
- Shadows are px and do not scale; restate them if they must change.
- **Convert against that ratio's canvas.** 16:9 and 1:1 share a 1080 height, so font sizes carry
  over but geometry does not; 9:16 needs new sizes (reusing a landscape size makes type ~1.8× too big).
- **Re-design, do not letterbox.** A split hero in 16:9 (text left, photo right) becomes a stack in
  9:16 and 1:1 (photo band on top, text beneath). A two-line headline in landscape may be three lines
  in portrait: give the box the height.
- Scene briefs per ratio live in `meta.description`, not `sizeProps`.
- A declared ratio with no overrides renders the base layout squeezed onto that canvas
  (`aspect.noOverrides`). Lay it out or do not declare it.
- Pick the base once. Changing it means rewriting every base value and every patch.

## Fonts

`fonts` maps every family any layer names (`style.font.family`, `familyAccent`, subtitles) to a
source. `weights` lists the weights to load; ask only for weights the face has (below), or the
browser fakes a bold.

**`builtin`** — the Fliki catalogue, by KEY (not display name):

| key | name | weights |
|---|---|---|
| `inter` | Inter | 100–900 |
| `montserrat` | Montserrat | 100–900 |
| `poppins` | Poppins | 100–900 |
| `roboto` | Roboto | 100, 300–900 |
| `lato` | Lato | 100, 300, 400, 700, 900 |
| `dmSans` | DM Sans | 400, 500, 700 |
| `workSans` | Work Sans | 100–900 |
| `spaceGrotesk` | Space Grotesk | 300–700 |
| `rubik` | Rubik | 300–900 |
| `raleway` | Raleway | 100–900 |
| `nunito` | Nunito | 200–900 |
| `lexend` | Lexend | 100–900 |
| `jost` | Jost | 100–900 |
| `barlow` | Barlow | 100–900 |
| `archivo` | Archivo | 100–900 |
| `bricolageGrotesque` | Bricolage Grotesque | 200–800 |
| `oswald` | Oswald | 200–700 |
| `bebasNeue` | Bebas Neue | 400 |
| `anton` | Anton | 400 |
| `archivoBlack` | Archivo Black | 400 |
| `unbounded` | Unbounded | 200–900 |
| `playFairDisplay` | Playfair Display | 400–900 |
| `fraunces` | Fraunces | 100–900 |
| `merriweather` | Merriweather | 300, 400, 700, 900 |
| `libreBaskerville` | Libre Baskerville | 400, 700 |
| `cormorant` | Cormorant | 300–700 |
| `instrumentSerif` | Instrument Serif | 400 |
| `robotoSlab` | Roboto Slab | 100–900 |
| `spaceMono` | Space Mono | 400, 700 |
| `robotoMono` | Roboto Mono | 100–700 |
| `caveat` | Caveat | 400–700 |
| `permanentMarker` | Permanent Marker | 400 |
| `notoSans` | Noto Sans | 100–900 |
| `notoSansJapanese` / `notoSansKorean` / `notoSansSC` | Noto Sans JP / KR / SC | 100–900 |
| `notoSansArabic` / `notoSansDevanagari` / `notoSansHebrew` / `notoSansThai` | Noto Sans … | 100–900 |

Keys are mostly the camel-cased name, with exceptions (`playFairDisplay`, `notoSansJapanese`).
`font.unknownBuiltin` means the key does not exist.

**`url`** — any other Google Font, by its exact CSS family name as the key:

```json
"fonts": { "Space Grotesk": { "kind": "url", "weights": [500, 700],
  "href": "https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;700&display=block" } }
```

Only `https://fonts.googleapis.com/css2?…` stylesheets; one href may list several families, and
each family key can share it. Never a font file (`.woff2`, `.ttf`).

**`custom`** (a font uploaded to Fliki) is refused by validation today (`font.storedRef`) — use a
`builtin` or `url` face. Never set `font-family` in css or `inlineStyle`. `italic` is a synthesized
slant of the regular face.

## Palettes

A palette makes the template's colours switchable: named **roles**, and 2–4 **variants** that give
every role a value. The editor, the template preview and videos made from the template switch
between them.

```json
"palette": {
  "roles": [ { "key": "ground", "note": "page background" }, { "key": "ink", "note": "type and plates" },
             { "key": "accent", "note": "rules and numerals" } ],
  "variants": [
    { "key": "paper", "name": "Paper", "colors": { "ground": "#F4F1EA", "ink": "#1B2433", "accent": "#D9573B" } },
    { "key": "ember", "name": "Ember", "colors": { "ground": "#FAF2E8", "ink": "#43231A", "accent": "#E0703C" } }
  ]
}
```

- `variants[0]` is the colours the package actually paints; `active` (optional) names the default.
- A switch replaces each role's value everywhere it appears — so every variant states every role,
  and no two roles share a value within a variant.
- Values are `#RRGGBB`, or `#RRGGBBAA` for a role that IS one tint (a 40% scrim). A 6-digit role
  also moves every tint of its colour (`#1B243380` follows `ink`, keeping its alpha).
- Write every colour in the package as hex — styles, markup, css, tweens. `rgb()`/`hsl()` never
  switch. Paint left outside the roles is listed by `palette.coverage`.
- Name roles by job (`ground`, `groundDark`, `ink`, `inkLight`, `muted`, `accent`), not hue.
- Author 2–3 alternates that differ in temperature or hue family (three navies read as a bug).
  Keep every contrast pair the design stacks near the first variant's: headline and body type on
  their ground around 10:1, secondary type ≥ 4.5:1, decorative accents ≥ 2:1 (≥ 4.5:1 if they
  carry type). A variant that breaks one pair ruins the scene nobody checked.
