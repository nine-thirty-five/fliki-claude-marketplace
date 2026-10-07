# Layers — types, slots, presenters, the common scene

The `type` decides what the user can do with a node later. A picture typed as a plain box can never
be replaced, cropped or trimmed; type every node for what it is.

## Choosing a type

| `type` | Use for | `data` |
|---|---|---|
| `text` | copy in one family and size | `{ content }` — HTML, see below |
| `generic` | a box that only paints (plate, scrim, divider, gradient), or copy mixing sizes or families | `{ tag: "div", html?, attrs?, classes?, inlineStyle? }` |
| `container` | a group that moves, clips, paints or fades as one | `{ tag: "div" }`; children set `parentId` |
| `media` | an image or video slot | `{ mediaMyId: "asset:<file>", loop?, volume?, speed? }` |
| `voiceover` | the scene's narration | `{ content }`, with `style: {}` |
| `audio` | music bed or sound | `{ mediaMyId, volume?, loop?, fade?: { in, out } }`, `style: {}` |
| `avatar` | a presenter | `{ mediaMyId, background: "remove", layout?, layoutFrame? }` |
| `vector` | icon or illustration | `{ tag: "svg", attrs: { viewBox }, html: "<path …/>" }` (inner SVG markup) |
| `shape` | a rectangle, circle or simple custom figure | `{ shape: "rectangle" \| "circle" }`, or `{ shape: "custom", path: "<path d=…/>" }` — inner SVG markup in a 100×100 viewBox, same markup rules as `vector` |

- The element decides: a lone `<img>` is a `media` layer, a lone `<svg>` a `vector`, audio an `audio`
  layer. A `generic` or `container` that is just one of those is refused (`layer.untyped`).
- Media refs are only ever `asset:<file>`. A template never writes `voiceId`, `avatarId`, `overlayId`
  or a raw id; a file writes those catalogue ids raw (see the files guide).
  `overlay` layers are not available in packages.
- `volume` is 0–100 (absent = 100); a music bed usually sits near 15–25.

## Text

- One family, size, weight and colour per layer. Emphasis inside it is markup in `content`:
  `<strong>`/`<b>` bold, `<em>`/`<i>` italic, `<u>` underline, `<span data-text-color="#E4572E">`
  coloured words, `<span data-color="#FFE45C">` a highlight behind words. Marks apply to whole words.
  `<br>` (or closing `</p>`) is a line break, and each line is a `line` part for motion.
- Words that differ in **size or family**: two `text` layers side by side (preferred), or one
  `generic` with `data.html` (renders as written, cannot be retyped).
- A text layer paints its own plate — `fill`, `radius`, `border`, `font.padding`, `shadow` — so a
  label on a pill is ONE layer, not a plate plus a label. `backgroundFit: "line"` fits the fill to
  each wrapped line (a highlighter look; no border then).
- **Height**: size the box for the worst case at `meta.maxLength` — lines × px size × line-height
  ratio, plus padding. Never `height: -1`.
- `font.alignY` defaults to `middle`; set `top` when copy should hang from the top.
- `font`: `family` (required), `weight`, `size`, `lineHeight`, `letterSpacing`, `color`,
  `align` (`left|center|right`), `alignY` (`top|middle|bottom`), `transform`
  (`none|uppercase|lowercase|capitalize`), `padding`, `backgroundFit` (`box|line`), `italic`
  (a synthesized slant), `underline`, `nowrap`, `familyAccent`, `shadow` (a hard poster extrusion).
- `stroke` on a text layer outlines the glyphs; `border` draws the box edge. `textShadow` lifts
  glyphs; `shadow` is the box's.

## Media slots

- **`fit: { "mode": "cover" }`** on every fillable picture: any photo then crops to fill. `contain`
  suits a logo. `fill` stretches whatever arrives and is flagged.
- **The box is the visible rectangle.** Do not oversize a picture and clip it with a parent frame;
  the next photo lands against a box nobody sees. Bias the crop with `fit.position: "30% 50%"`.
- A slow push (Ken Burns) is `style.crop`, never a tween: insets in % of the box, `at` a 0–1
  fraction of the layer's time on screen.
  ```json
  "crop": [ { "at": 0, "left": 0, "right": 0, "top": 0, "bottom": 0 },
            { "at": 1, "left": 3, "right": 3, "top": 3, "bottom": 3 } ]
  ```
- A `radius` on a media layer clips the picture to its corners; no wrapper needed.

### Which slots are b-roll

`meta.broll: true` turns a slot into a SEQUENCE of shots timed to the narration; unmarked, it holds
one picture however long the scene talks. It is the only way videos from the template get cutaways.

- Mark the scene's main picture — full-bleed, a large band, a background — in a scene that narrates.
- Hold grid tiles, portraits beside a name, logos, small insets, title cards, quote cards, and any
  scene with `meta.silent`.
- One marked slot per scene at most. When unsure, hold. Judge the role, not `customDuration`.

## Narration

- One `voiceover` per scene at most: `orderKey: "z"`, `style: {}`, `meta.slotKey: "script"`, and a
  placeholder of one or two plain sentences saying what that scene says.
- A scene with nothing to say (logo sting, interstitial, end card) gets no voiceover and
  `meta.silent: true`. Never both. A `video` or `audio` template must narrate somewhere.
- Fliki assigns a preview voice at create; the person using the template picks their own.
- On create, narrated scenes drop `customDuration`: the real script sizes them. Silent scenes keep it.
- Captions are optional: a `subtitle` on the voiceover sets their box and look for this layout.
  Omit it unless the design reserves a caption band.
  ```json
  "subtitle": { "data": { "reveal": "word", "animation": "pop" },
    "style": { "left": 10, "top": 80, "width": 80, "height": 10,
      "font": { "family": "montserrat", "size": 30, "weight": 700, "color": "#FFFFFF", "align": "center" } } }
  ```
  `reveal`: `word|phrase|sequence|full`; `animation`: `none|fade|rise|pop|bounce|slide|zoom|blur`.

## Presenters (`avatar`)

Use one only where the box makes sense only as a person: a cut-out presenter in a composed layout,
a portrait disc beside a name and role, someone talking to camera. A workflow without a presenter
drops the layer entirely, so a picture that also works as a picture stays `media`. Ask when unsure.

- `data.mediaMyId` is the placeholder portrait shown until a presenter is chosen (ideally on a
  transparent background); `data.background: "remove"` always, so the presenter plays cut out.
- `meta.slotKey`, `meta.prompt` (direction for the presenter) and a `meta.description` of what the
  scene does without one. One avatar per scene.
- Never on an avatar: `subtitle`, `data.avatarId`, `meta.broll`, `reserved`, `reservedCamera`,
  `timingPercent`, `removeBackground`, `ignoreCopy`, `timing.cuts`, `timing.mute`.
- **In a disc**: `data.layout: "circle"`, a box square in px (on 16:9, `height = width × 16/9`),
  `radius: 100`, `data.layoutFrame` with the rectangle it replaces, and no `crop` or `fit.zoom`.
  ```json
  "host-disc": { "type": "avatar", "orderKey": "c",
    "data": { "mediaMyId": "asset:host.webp", "background": "remove", "layout": "circle",
      "layoutFrame": { "top": 18, "left": 8, "width": 24, "height": 64, "radius": 0 } },
    "style": { "top": 32, "left": 11.5, "width": 16.88, "height": 30, "radius": 100, "fit": { "mode": "cover" } },
    "meta": { "slotKey": "presenter", "description": "The host beside their name; a name card without one." } }
  ```

## Containers

- `parentId` names a `container` in the same scene, nothing else. Children still store canvas %.
- Keep a container only when it paints (`fill`, `radius`, `border`, `shadow`), moves or turns as one
  (an `angle`, or a tween aimed at it), clips on purpose (containers clip; `overflow: "visible"`
  lets children out), or fades/blurs as one. Otherwise flatten it.
- No `style.layout` (flex): place every child yourself.

## The common scene

`common` is shown with every scene. `style.backgroundColor` there is the file's background.

- `orderKey` below `"V"` (`1`–`9`, `A`–`U`) paints **behind** every scene; above (`W`–`Z`, `a`–`z`)
  **in front**. Never `"V"`. A full-bleed backdrop goes below `V`, or it hides the whole video.
- A persistent logo: `orderKey: "W"`, `timing: { "skipFirstScene": true }`, `meta: { "ignoreCopy": true, "locked": true }`.
- A music bed spanning the video is an `audio` layer here, faded with `data.fade`, not tweens.
- No `transition` or `customDuration` on `common`.
- Common layers sandwich the scenes: behind (below `V`) and in front (above `V`). Their
  `timing.in`/`out` are seconds from the file start, so one can span several scenes; the files
  guide covers cross-scene staging. Never a voiceover or avatar here.

## The fill contract (`meta`)

Scene:

| field | effect |
|---|---|
| `slotKey` | makes the scene choosable as a layout — give every scene one |
| `description` | per-ratio brief for filling it: `{ "16:9": "…", "9:16": "…" }` |
| `silent` | deliberately not narrated |
| `keywords` | up to 20 selection hints |

Layer:

| field | effect |
|---|---|
| `slotKey` | fillable, and its name; unique within the scene |
| `maxLength` | character budget for generated copy |
| `description` | what to write or show here (≤500 chars) |
| `broll` | this picture is a shot sequence |
| `removeBackground` | cut the subject out of whatever fills the slot |
| `locked` | content replaceable, geometry and style fixed |
| `reserved` | keep the layer even if its fill comes back empty |
| `ignoreCopy` / `aiLock` | leave this node alone (an ordinal, a logo) |
| `note` | the name shown in the Layers panel (≤120 chars) |
| `prompt` | direction for generating this slot |

Placeholder copy is generic and short — "Your headline here", not a finished claim; over 120
characters (or `maxLength`) is flagged. A layer without `slotKey` keeps its placeholder forever.

## Scene css

Only `scene.css` paints (a layer's own `css` is ignored). Address a layer by `data.attrs.id` →
`#id`. Prefer typed fields first: gradients are a `fill`, edges a `border`, shadows `shadow`,
clipping `overflow`. No `@keyframes`, `@media` or `font-family` in css.
