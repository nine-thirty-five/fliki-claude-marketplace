---
name: fliki-files
description: Author a finished Fliki video, audio file or design directly — every scene, layout, headline, picture, clip, narration line, caption, music bed and animation written by you as a playback package and saved with file_create — or read any of the user's Fliki files (made here, by a workflow or in the editor) and change it scene by scene with file_read and file_update. The default build for composed videos — explainers, ads and promos, stories, documentaries, motion graphics, presenter-led and social videos — including generating and waiting for AI clips, presenter lip-sync and narration before export, and cross-scene elements staged in the common scene. Also use to change specific scenes, copy, timing, media or motion in an existing Fliki file.
---

# Fliki files

A file is the same JSON **package** a template is (see the fliki-templates guide): scenes made of
layers, every position, size, colour and tween written exactly as Fliki stores it. Fliki renders
what you write, number for number. The difference is that a file is the finished video: real copy,
real narration, a real voice — nothing waits to be filled.

| Tool | Does |
|---|---|
| `template_units` | px you designed → stored values. Use it for every number. |
| `file_create { package, assets?, voiceId?, languageId?, dialectId? }` | Saves a new file → `fileId`, `editorUrl`, `revision`. |
| `file_read { fileId, scenes? }` | Any file as a package + `outline` of every scene + `revision`. |
| `file_update { fileId, revision, scenes?, order?, common?, fonts?, palette?, name?, assets?, voiceId? }` | Changes scenes in place. |
| `file_generate { fileId, wait? }` | Synthesizes the narration, lip-syncs every presenter, and returns each scene's `start`/`duration`. |
| `export_file`, `get_file` | Render it and fetch the download. |
| `generate_image`, `generate_video`, `generate_music`, `upload_assets`, `import_assets`, `list_assets` | The media it uses. |
| `transcribe_asset { id }` | What a video or audio asset says, each word with `start`/`end` seconds. |

Creating and updating are free. `file_generate` spends credits for the narration and presenter
seconds it makes, and `export_file` for the render (plus any narration still missing). Voices and presenters marked `locked` in `list_voices` or
`list_characters` need a higher plan. A package or update is at most 2 MB.

## Create

1. **Storyboard.** Storyboard a multi-scene piece and get it approved first (fliki-storyboard),
   then build to the approved version. Skip this only when the user said to go straight to
   creation. The brief
   settles these, and you ask only for what is missing:
   - the format: `video` (the default), `audio` or `design`;
   - one aspect ratio, unless the user wants several: `16:9` 1920×1080, `9:16` 1080×1920,
     `1:1` 1080×1080;
   - the message of each scene;
   - the look: a brand kit from `list_brand_kits` when the user has one (below), otherwise colours
     and fonts you choose.
2. **Voice and language.** `list_voices` → `voiceId`, passed once to `file_create` (every voiceover
   without its own `voiceId` gets it). The language defaults to the voice's; pass `languageId` +
   `dialectId` (from `list_languages`) to override. A scene can name its own `data.voiceId`; a
   two-voice exchange is `data.speakers: [{ name, voiceId }, …]` with turns in `content`
   ("Maya: … Sam: …").
3. **Media.** Every picture, clip and music bed is a user asset: `generate_image`, `generate_video`,
   `generate_music` (these spend credits — say so first), `list_assets` (already in the library:
   check it before uploading), `upload_assets` (the user's own files) or `import_assets` (public
   links).
   A generation that returns a `handle` (every clip, every track, some images) is not an asset yet:
   call `get_generation { handle, wait: true }` until it has `succeeded`, and only then use its
   asset id. Start independent generations together, then wait on each.
   For a recurring character, pass the character's `referenceAssetIds` (from
   `list_characters`, source `mine`) or an approved reference image as `referenceAssetIds` on every
   shot, so the face and wardrobe hold.
   Name each one in the package as `"asset:<name>"` (letters, digits, `._-/`), declare it under
   `assets` with its `kind`, and map the names in the call: `assets: { "<name>": "<assetId>" }`.
4. **Plan in px, convert with `template_units`** — geometry is % of the canvas, type an
   exponential 1–100 scale; a guessed font size is routinely 2–3× off.
5. **Write** the package — shape in the templates guide, layer types in the `layers` reference,
   units in `style`, motion in `motion`.
6. **`file_create`.** Errors come back as `{ created: false, errors }` and nothing is written — fix
   and call again. Share `editorUrl`: the user can watch it in Fliki straight away.
7. **Generate and wait.** `file_generate { fileId, wait: true }` synthesizes every narration line,
   then lip-syncs every `avatar` layer to its scene's narration, and holds until the presenters
   finish (call it again if it returns while one is still `generating`; it never restarts work).
   Its `outline` gives each scene's real `start` and `duration` in seconds, the voiceover state and
   the avatar state. `errors` names any presenter that could not start.
8. **Place timing-dependent layers** now that narration has fixed the timing: common-scene spans
   (below) and anything else keyed to absolute seconds, with `file_update`.
9. **Export** with `export_file`, then `get_file { id, wait: true }` for `export.downloadUrl`.
   `export_file` refuses while a presenter has no lip-synced clip; run `file_generate` first.

## Edit (any file)

1. `file_read { fileId }`. A small file comes back whole; a large one returns the head, the
   scenes that fit, and `omitted` — ask for those with `scenes: [...]`. `outline` lists every scene
   with its duration, layer count and first words of narration, so read only what you will change.
2. Send only what changes: `file_update { fileId, revision, scenes: [<edited scene>] }`.
   - A scene with an existing id replaces that scene; a new id is added at the end.
   - `order` (optional) is the complete list of scene ids: it reorders, and any scene left out of
     it is deleted.
   - Keep every id you did not mean to change. Unchanged narration and presenter clips are kept;
     a changed script or voice is regenerated (and billed) on the next export.
   - New media: add the name to `assets`. Names from `file_read` are already known.
3. Every update returns a new `revision` — use it for the next one. `"The file changed"` means the
   user edited it or an export generated narration since your read: `file_read` again.
4. A backup is taken before each update, so the user can restore the previous version from the
   editor's Backups.

Ids in a file you did not create are its stored ids (24 hex characters). Keep them exactly.

## How a file differs from a template

- **No fill contract.** No `slotKey`, `maxLength`, placeholders, `meta.silent` or palette needed.
- **Catalogue ids are written raw**: `voiceId`, `voiceStyleId`, `speakers[].voiceId`, `avatarId`
  (from `list_characters`), and a custom font's `fontId`. Media is always `asset:<name>`.
- **Narration sets the length.** Leave `customDuration` off narrated scenes — a pin outranks the
  narration and cuts a longer script off. Set it on silent scenes (end cards, stings).
- **Captions** are the voiceover's `subtitle`: `{ "data": { "reveal": "word", "animation": "pop" },
  "style": { box + font } }`, with `reveal` `word|phrase|sequence|full`. Start from a Fliki caption
  look: `list_presets { kind: "subtitle", aspectRatio }` returns each one as a ready `subtitle`
  (sized for that ratio). Use it as is and add only the box (`left`, `top`, `width`, `height`);
  recolour `font.color` or `caption.highlight`/`caption.keyword` to the brand if needed.
- **Motion is yours to write.** Author every animation as GSAP tweens (`layer.gsap`,
  `scene.gsap`, common-scene layers) and choreograph it to the story; that is where the craft is.
  Fliki's animation presets are only for when the user asks for one by name: then set
  `"meta": { "animation": { "preset": "<key>" } }` on that layer with a key from
  `list_presets { kind: "animation" }` (`text` keys on text, `visual` keys on media, avatar and
  shape). A preset replaces that layer's own tweens.
- **Music** spanning the video is an `audio` layer in `common` with `orderKey` below `"V"`,
  `volume` 15–25 and `data.fade`.
- **Footage cut to what is said**: `transcribe_asset` the clip, then keep the lines you want as
  `timing.cuts: [{ "start", "end" }]` on its media layer — word times are the clip's own seconds.
- **Cutaways timed to the words**: a media layer with `data.generate.reference` set to the exact
  phrase it illustrates gets its window from the narration on export.
- **Presenters**: an `avatar` layer with `data.avatarId` (from `list_characters { type: avatar }`)
  and `data.background: "remove"`, one per scene, in a scene with narration. `file_generate`
  lip-syncs it to that narration. Changing a narrated scene's script clears its presenter clip;
  run `file_generate` again before exporting.

## Brand kits

`list_brand_kits` returns the user's kits and their team's shared ones. When the user has one (ask
which, if several), build the file from it:
- `colors` → the grounds, ink and accent; put them in `palette` roles so the user can swap them.
- `fonts[].family` are catalogue keys: declare them in the package's `fonts` and use the first for
  headings, the second for body (with each font's `color` where given).
- `logo.assetId` → a media layer `asset:logo` in the common top bun (`orderKey: "W"`,
  `timing.skipFirstScene`), and on the cover or end card.
- `voice.id` → the `voiceId` for `file_create`.
- `designGuide` and `about` say how the brand looks and talks: follow them for layout, type,
  motion and copy.

## The common scene — the bun around every scene

`common` plays for the whole file, so its layers sandwich the scenes like the two halves of a bun
around the filling:

- **Bottom bun** — `orderKey` below `"V"` (`1`–`9`, `A`–`U`) paints behind every scene: a
  continuous backdrop, texture, gradient or slow shape field that keeps cuts seamless. It shows
  only through scenes that leave their `style.backgroundColor` off (the `background.sceneMissing`
  warning is expected there) or paint a translucent one.
- **Top bun** — `orderKey` above `"V"` (`W`–`Z`, `a`–`z`) paints in front of every scene: a logo,
  frame, progress bar, a title or graphic that stays put while the scene under it changes.
- Never `"V"` itself, and never an opaque full-bleed layer above it — that hides the video.

Use it whenever an element has to live **across a scene boundary** — text, media, shapes,
graphics — which a stock `transition` can't do:
- a headline or label that holds across two or three scenes while their content changes;
- a shape, image or mask that travels from one scene into the next, or wipes across the cut as a
  custom transition;
- a picture that starts in one scene and keeps playing behind the next;
- a running element (counter, map, timeline bar) that advances through several scenes.

Timing:
- A common layer's `timing.in` and `timing.out` are **seconds from the start of the file**, not of
  a scene. Span scenes 2–3 with `in` = scene 2's `start` and `out` = scene 3's
  `start + duration` from `file_generate`'s (or `file_read`'s) outline. Absent `out` = to the end.
- Its `layer.gsap` runs on its own clock, starting at its `timing.in`, so tween positions inside
  the span the same way as in a scene.
- `timing.skipFirstScene` / `skipLastScene` keep a persistent layer off the cover and the end card.
- Narration sets scene lengths, so place spans after `file_generate`, and place them again after
  any script change.

Keep out of `common`: voiceover and avatar layers (they belong to their scene). Audio is fine: a
music bed, an ambience, a sound effect at an absolute time.

## Rules that decide quality

- One job per scene; 4–10 scenes for a short video.
- Every scene paints its ground (`style.backgroundColor` or a full-bleed picture) unless a common
  backdrop is meant to show through it, and `common.style.backgroundColor` is the file's.
- Motion: transforms, opacity and filter only; `cw`/`ch` distances; nothing timed from the scene's
  end — narration decides it. Use `transition` between scenes.
- Colours as 6/8-digit hex. Fonts by catalogue key (`montserrat`, `playFairDisplay`).
- The safety rules in the templates guide apply unchanged (allowed tags, attributes, css urls).
  A `fliki-media:` ref is media already in the file: keep it exactly as read. Add new media with
  `asset:`.

## Example — creates clean

```json
{
  "format": "playback-v2-template", "revision": 1, "name": "Better coffee in 30 seconds",
  "settings": { "format": "video", "aspectRatio": "16:9" }, "aspectRatios": ["16:9"],
  "fonts": { "montserrat": { "kind": "builtin", "weights": [600, 800] } },
  "assets": { "beans.webp": { "kind": "image" }, "bed.mp3": { "kind": "audio" } },
  "common": { "id": "common", "style": { "backgroundColor": "#14110F" }, "layers": {
    "music": { "type": "audio", "orderKey": "B", "style": {}, "meta": {},
      "data": { "mediaMyId": "asset:bed.mp3", "volume": 18, "loop": true, "fade": { "in": 1, "out": 2 } } } } },
  "scenes": [
    { "id": "hook", "name": "Hook", "transition": { "kind": "fade", "duration": 0.5 },
      "style": { "backgroundColor": "#14110F" }, "meta": {},
      "layers": {
        "hook-photo": { "type": "media", "orderKey": "a", "data": { "mediaMyId": "asset:beans.webp" },
          "style": { "fit": { "mode": "cover" }, "filters": { "brightness": -25 } }, "meta": {},
          "gsap": { "tweens": [{ "target": { "part": "media" }, "op": "fromTo", "at": 0, "duration": 6,
            "fromVars": { "scale": 1.08 }, "vars": { "scale": 1 }, "ease": "none" }] } },
        "hook-title": { "type": "text", "orderKey": "b", "data": { "content": "Better coffee<br>in 30 seconds" },
          "style": { "left": 6.25, "top": 30.556, "width": 55, "height": 27.778,
            "font": { "family": "montserrat", "size": 63, "weight": 800, "lineHeight": -12,
              "color": "#FFF4E6", "align": "left", "alignY": "top" } },
          "meta": {},
          "gsap": { "tweens": [{ "target": { "part": "line" }, "op": "from", "at": 0.2, "duration": 0.7,
            "vars": { "yPercent": 50, "opacity": 0 }, "ease": "power4.out", "stagger": 0.1 }] } },
        "hook-vo": { "type": "voiceover", "orderKey": "z", "style": {}, "meta": {},
          "data": { "content": "Want better coffee at home? It takes thirty seconds and one change." },
          "subtitle": { "data": { "reveal": "word", "animation": "pop" },
            "style": { "left": 10, "top": 82, "width": 80, "height": 9,
              "font": { "family": "montserrat", "size": 30, "weight": 600, "color": "#FFFFFF", "align": "center" } } } } } },
    { "id": "end", "name": "End card", "customDuration": 3, "style": { "backgroundColor": "#14110F" }, "meta": {},
      "layers": {
        "end-line": { "type": "text", "orderKey": "a", "data": { "content": "Grind fresh. Every time." },
          "style": { "left": 10, "top": 42, "width": 80, "height": 16,
            "font": { "family": "montserrat", "size": 50, "weight": 800, "color": "#FFF4E6",
              "align": "center", "alignY": "middle" } },
          "meta": {},
          "gsap": { "tweens": [{ "target": { "self": true }, "op": "from", "at": 0.1, "duration": 0.6,
            "vars": { "y": "3ch", "opacity": 0 }, "ease": "power3.out" }] } } } }
  ]
}
```

Create with `voiceId: "<id from list_voices>"` and
`assets: { "beans.webp": "<image asset id>", "bed.mp3": "<music asset id>" }`.

## References

The templates guide's references apply to files too: `layers`, `style`, `motion`, `validation`.
Without skills, `get_guide { topic: "files", reference: "<name>" }` returns them.
