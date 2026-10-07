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
| `kind` | `none`, `blur`, `clock`, `fade`, `flip`, `slide`, `wipe`, `zoom` |
| `direction` | `slide`/`flip`: `fromLeft`, `fromTop`, `fromRight`, `fromBottom`; `wipe`: those plus `fromTopLeft`, `fromTopRight`, `fromBottomRight`, `fromBottomLeft`; `zoom`: `in`, `out` |
| `duration` | seconds — 0.5 short, 1 medium, 1.5 long |
| `ease` | `linear`, `spring` |
| `sfx` | `none`, `click`, `bell`, `boom`, `pop`, `whoosh` |

Other names (`dissolve`, `push`, `circle-wipe`) are refused: map `dissolve` → `fade`, a push →
`slide`. The transition does not eat into the scene's narration.

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
