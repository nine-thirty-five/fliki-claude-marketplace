# Motion — tweens, clocks, transitions

Motion is GSAP written as JSON: a list of tweens, no code. The same frame always renders the same,
which is what lets Fliki seek, preview and export it. `design` templates ignore motion and
transitions.

## Where a program lives

| program | clock starts at | may target |
|---|---|---|
| `layer.gsap` | that layer's `timing.in` (0 when absent) | `{ "self": true }`, `{ "part": … }`, `{ "css": … }` |
| `scene.gsap` | the scene's start | `{ "layerId": "<id in this scene>", "part"?: … }`, `{ "css": … }` |
| `subtitle.gsap` | its layer's start | `{ "self": true }`, `{ "part": … }` |

Prefer `layer.gsap` with `self`. Use `scene.gsap` when one choreography spans several layers, or to
move a container as a group.

```json
"gsap": {
  "labels": { "beat": 0.4 },
  "tweens": [
    { "target": { "self": true }, "op": "from", "at": "beat+=0.2", "duration": 0.8,
      "vars": { "x": "-6cw", "opacity": 0 }, "ease": "power3.out" }
  ]
}
```

Tween fields: `target`, `op` (`to|from|fromTo|set`), `at` (seconds on the owner's clock, or a
declared label with `+=`/`-=`), `duration`, `vars`, `fromVars` (for `fromTo`), `ease`, `stagger`
(a number, or `{ each | amount, from: start|center|end|<index>, grid? }`), `repeat`, `yoyo`,
`immediateRender`. Timeline fields: `tweens`, `labels`, `defaults`, `offset`. Values in `vars` are
numbers or strings only.

## Parts

| layer type | parts |
|---|---|
| `text` | `line`, `word`, `char`, `row`, `motion` |
| `media`, `avatar` | `motion`, `media` (the picture inside its frame) |
| everything else | `motion` |

A part the layer does not have matches nothing and the tween silently does nothing
(`gsap.unknownPart`). `word`/`char` split the text only when targeted.

## Presets (only on request)

Write motion as tweens. Only when the user names a Fliki animation preset, set
`"meta": { "animation": { "preset": "<key>" } }` on that layer with a key from
`list_presets { kind: "animation" }` (`text:*` keys on text, the rest on media, avatar and shape).
It replaces that layer's own `gsap` tweens.

## The rules

1. **Transforms, opacity and filter only**: `x`, `y`, `xPercent`, `yPercent`, `scale`, `scaleX`,
   `scaleY`, `rotation`, `skewX`, `skewY`, `opacity`, `autoAlpha`, `filter`, `transformOrigin`.
   Never `left`, `top`, `right`, `bottom`, `width`, `height`, `fontSize`, `letterSpacing`,
   `wordSpacing`, `lineHeight`, `margin*`, `padding*`, `roundProps` — they stutter in export
   (`gsap.reflowTween`). Position, size and tracking are static `style`.
2. **Distances in canvas units**: `"x": "-15.6cw"` (% of canvas width), `"y": "4ch"` (% of canvas
   height), so a move means the same at every ratio. `xPercent`/`yPercent` are relative to the
   element itself ("rise by half its height").
3. **No endless loops.** `repeat: -1` is refused. For ambient motion compute a finite count,
   `repeat = ceil(seconds / cycle) - 1`, with `yoyo: true` so it comes to rest where it started.
4. **Chains on one target**: the first `from`/`fromTo` sets the opening state; every later
   `from`/`fromTo` on the same target needs `"immediateRender": false`, or frame 0 shows the middle
   of the animation. `to` tweens need nothing.
5. **Nothing timed from the scene's end.** Narration decides scene length, and the script someone
   writes into the template is longer or shorter than yours — narrated scenes lose
   `customDuration` on create. An exit at 4.6s of a 5s scene fires a third of the way into a 14s
   one, leaving an empty frame under the narration. So: entrances only, the scene holds its final
   state, no fade-outs near the end, no `timing.out` in the last stretch (absent `out` = until the
   scene ends). Soften the boundary with `transition`, which lands wherever the end turns out to be.
6. **No timers, randomness or text changes.** Vary with `stagger`. Tweens may never set
   `innerHTML`, `outerHTML`, `innerText`, `textContent`, `text`, `attr` or `on*`, so there are no
   counting numbers.
7. **Fit the authored length.** A tween ending after `customDuration` is flagged
   (`gsap.overrun`); keep entrances inside the first seconds.
8. **Two things are not tweens**: a Ken Burns push is `style.crop` keyframes, and audio fades are
   `data.fade: { in, out }` (seconds, applied at the layer's real start and end).

## Transitions

`scene.transition` is the OUTGOING boundary into the next scene. The last scene has none.

```json
"transition": { "kind": "wipe", "duration": 0.6, "direction": "fromLeft", "ease": "linear", "sfx": "whoosh" }
```

| field | values |
|---|---|
| `kind` | `none`, `blur`, `clock`, `fade`, `flip`, `slide`, `wipe`, `zoom`, `custom` (with `spec`) |
| `direction` | `slide`/`flip`: `fromLeft`, `fromTop`, `fromRight`, `fromBottom`; `wipe`: those plus `fromTopLeft`, `fromTopRight`, `fromBottomRight`, `fromBottomLeft`; `zoom`: `in`, `out` |
| `duration` | seconds — 0.5 short, 1 medium, 1.5 long |
| `ease` | `linear`, `spring` |
| `sfx` | `none`, `click`, `bell`, `boom`, `pop`, `whoosh` |

Other names (`dissolve`, `push`, `circle-wipe`) are refused: map `dissolve` → `fade`, a push →
`slide`. The transition does not eat into the scene's narration.

### Custom transitions

Prefer a named kind. When none gives the look asked for (a diagonal wipe with a blur, an iris,
blinds), use `kind: "custom"` and describe it in `spec`; `spec` is ignored on any other kind.

```json
"transition": { "kind": "custom", "duration": 0.8, "spec": { "name": "Blur wipe",
  "enter": { "look": { "reveal": { "shape": "linear", "angle": 135, "amount": 0, "feather": 8 }, "blur": 2 } } } }
```

Both scenes are flat pictures. `enter` animates the incoming scene FROM its `look` to normal;
`exit` animates the outgoing scene from normal TO its `look`. A track takes an optional `ease` (GSAP
name, default `power2.inOut`) and `keys` — stops with `at` strictly between 0 and 1. The incoming
scene paints on top unless `top: "exit"`. `plates` are solid colour panels
(`{ color, layer?: "over" | "between", keys: [{ at, x, y, w, h, ease? }] }`) that need keys at 0 and
1 and must be off canvas or transparent at both. `name` labels it in the editor.

Look fields — positions and sizes % of the canvas, angles degrees:
- `opacity` 0..1 · `x`, `y` · `scale`, `scaleX`, `scaleY` · `rotate` · `rotateX`, `rotateY` (3D;
  `perspective` px) · `skewX`, `skewY` · `originX`, `originY` (default 50)
- `blur` 0..4 (% of the short side) · `brightness`, `contrast`, `saturate` (1 = unchanged) ·
  `grayscale` 0..1 · `hue` degrees · `tint: { color: "#hex", amount: 0..1 }`
- `reveal` — the visible part of the scene, `amount` 0..1 visible:
  - `{ shape: "linear", angle, amount, feather? }` — a wipe; `angle` is the way the edge travels
    (90 left→right, 180 top→bottom)
  - `{ shape: "radial", cx, cy, r, feather?, invert? }` — an iris; `r` % of the diagonal (100
    covers the frame); `invert` shows the scene outside the circle
  - `{ shape: "inset", top, right, bottom, left, round? }` — a rectangle
  - `{ shape: "polygon", points: [{ x, y }, …] }` — same point count in every key
  - `{ shape: "bands", axis: "x" | "y", count, amount, stagger? }` — blinds
  - `{ shape: "cells", cols, rows, amount, stagger?, order?: "rows" | "diagonal" | "center" | "random" }`

More specs:

```
{ "name": "Iris close", "enter": { "look": { "reveal": { "shape": "radial", "cx": 50, "cy": 50, "r": 100, "invert": true } } },
  "exit": { "look": { "scale": 0.9 } } }
{ "name": "Blinds", "enter": { "look": { "reveal": { "shape": "bands", "axis": "y", "count": 8, "amount": 0, "stagger": 0.4 } } } }
```

Not possible: per-pixel effects (pixelate, ripple, page curl) or a scene breaking apart (shatter).

## Patterns

Headline lines rising in:

```json
{ "target": { "part": "line" }, "op": "from", "at": 0.3, "duration": 0.7,
  "vars": { "yPercent": 50, "opacity": 0 }, "ease": "power4.out", "stagger": 0.1 }
```

Words popping in:

```json
{ "target": { "part": "word" }, "op": "from", "at": 0.2, "duration": 0.5,
  "vars": { "scale": 0.6, "opacity": 0 }, "ease": "back.out(2)", "stagger": 0.05 }
```

A rule drawing across from the left:

```json
{ "target": { "self": true }, "op": "fromTo", "at": 0.9, "duration": 0.5,
  "fromVars": { "scaleX": 0 }, "vars": { "scaleX": 1, "transformOrigin": "0% 50%" }, "ease": "power3.out" }
```

Soft focus-in:

```json
{ "target": { "self": true }, "op": "from", "at": 0, "duration": 0.9,
  "vars": { "opacity": 0, "scale": 1.04, "filter": "blur(12px)" }, "ease": "power2.out" }
```

Cards arriving one after another (scene program, one tween per card):

```json
"gsap": { "tweens": [
  { "target": { "layerId": "card-1" }, "op": "from", "at": 0.3, "duration": 0.6, "vars": { "y": "5ch", "opacity": 0 }, "ease": "power3.out" },
  { "target": { "layerId": "card-2" }, "op": "from", "at": 0.45, "duration": 0.6, "vars": { "y": "5ch", "opacity": 0 }, "ease": "power3.out" },
  { "target": { "layerId": "card-3" }, "op": "from", "at": 0.6, "duration": 0.6, "vars": { "y": "5ch", "opacity": 0 }, "ease": "power3.out" }
] }
```

A gentle float after the entrance (ends at rest):

```json
{ "target": { "self": true }, "op": "to", "at": 1.3, "duration": 1.5,
  "vars": { "y": "-0.8ch" }, "ease": "sine.inOut", "yoyo": true, "repeat": 1 }
```

A photo easing in while a slow push runs inside it: an `opacity` `from` tween on the layer plus
`style.crop` keyframes (see layers). The crop moves the picture; the tween moves the box.

Keep each scene to a few purposeful entrances — a headline, its support, the picture — staggered
by 0.1–0.3s, finished within about 1.5s.
