---
name: fliki-templates
description: Design and save the user's own reusable Fliki template — a multi-scene video, audio or design layout (cover, points, stats, quotes, presenter, end card) with fillable headline, image, b-roll and narration slots, brand colours and fonts, laid out for 16:9, 9:16 and 1:1 — by writing a template package and saving it with template_units, template_validate and template_create. Use when the user asks to create, design, build, recreate or brand a Fliki template (from a brief, a style they describe, a website or a design), to turn a look into a reusable layout for script-to-video or url-to-video, or to change, fix or restyle a template they made.
---

# Fliki templates

A template is one JSON **package**: scenes made of layers, every position, size, colour and tween
written exactly as Fliki stores it. Fliki renders what you write, number for number. When someone
makes a video from it, Fliki fills the **slots** you declared (headline, image, narration…) and
keeps the rest as designed.

| Tool | Does |
|---|---|
| `template_units` | px you designed → stored values. Use it for every number. |
| `template_validate { package }` | Schema, layout and safety checks → `{ valid, errors, warnings }`. Writes nothing. |
| `template_create { package, assets?, replace? }` | Saves it → `templateId`, `fileId`, `editorUrl`. |
| `generate_image`, `upload_assets`, `import_assets`, `list_assets` | The pictures, clips and music it uses. |

Up to 100 templates per user; a package is at most 2 MB. Voices marked `locked` need a higher plan.

## Procedure

1. **Brief.** Ask only what is missing; propose defaults. Format: `video` (default), `audio`
   (narration and music, nothing painted) or `design` (stills; motion ignored, no narration
   needed). Base aspect ratio plus others — `16:9` 1920×1080, `9:16` 1080×1920, `1:1` 1080×1080;
   declare only ratios you will lay out. 3–8 scenes, one job each (cover, point + picture, stat,
   quote, list, presenter, end card). Brand colours (hex), fonts, mood — from a site or design the
   user shares, take its palette, type and layout.
2. **Plan** each scene in px on the base canvas: id, message, every layer's box, whether it
   narrates, which picture becomes b-roll. Copy is short generic placeholder text.
3. **Media.** Every picture, clip or music bed is a user asset: `generate_image { prompt,
   aspectRatio }` (spends credits — say so; a `handle` means poll `get_generation`),
   `upload_assets` (the user's own files), `import_assets` (public https) or `list_assets`. Keep
   `id`, `type`, and `duration` for video/audio. The package names each file locally
   (`"asset:cover.webp"`, letters/digits/`._-/`) and declares it under `assets` with the same
   `kind`.
4. **Convert every number** with `template_units` — never estimate. Geometry is % of the canvas,
   type is an exponential 1–100 scale, radius a fraction of half the box's shorter side; a guessed
   font size is routinely 2–3× off.
   ```json
   { "canvas": { "width": 1920, "height": 1080 }, "box": { "width": 820, "height": 300 },
     "values": { "left": 120, "top": 330, "width": 820, "height": 300,
                 "fontSize": 88, "lineHeight": 1.05, "letterSpacing": -1.5, "radius": 16 } }
   ```
   Store each row's `stored`; `expressible: false` means change the design. Convert every other
   ratio against **its own** canvas. See [references/style.md](references/style.md).
5. **Write** the package (below; layer types in [references/layers.md](references/layers.md),
   motion in [references/motion.md](references/motion.md)).
6. **Validate until clean**: `template_validate`, fix, repeat — warnings too unless you know why one
   is fine. Codes and fixes: [references/validation.md](references/validation.md). Validation cannot
   see composition (dark type on a dark plate, a headline wrapping into a photo): re-check each ratio.
7. **Create**: `template_create { package, assets: { "cover.webp": "<assetId>", … } }`, every
   `asset:` file mapped. Share `editorUrl` — the user previews ratios and palettes in Fliki. Its
   warnings are informational (narrated scenes resized by narration, a preview voice assigned).
8. **Change**: edit the same package, validate, `template_create` with `replace: "<templateId>"`.
   Keep ids stable — matched layers keep their voice, presenter and generated narration (until
   their script changes); a renamed id is new, a missing one is deleted. Edits made in the Fliki
   editor are overwritten, so ask first. The description is only set on first create.
9. **Use**: `list_templates { source: 'mine' }`, then `templateId` in `start_workflow`
   (`script-to-video`, `url-to-video`).

## The package

| Field | Holds |
|---|---|
| `format`, `revision` | `"playback-v2-template"`, `1` |
| `name`, `description?`, `useCases?`, `category?` | name ≤120 chars; description plain markdown |
| `settings` | `{ format: video\|audio\|design, aspectRatio: <base> }` |
| `aspectRatios` | every supported ratio, base included |
| `fonts` | `{ "<family>": { kind: builtin\|url, weights? } }` for every family used |
| `palette?` | colour roles and 2–4 switchable variants |
| `assets?` | `{ "<file>": { kind: image\|video\|audio, duration?, name? } }` — duration required for video/audio |
| `common?` | scene `"common"`: file background, layers shown across every scene |
| `scenes` | 1–80 scenes, in playback order |

Scene: `id`, `name`, `customDuration` (s), `transition?`, `style.backgroundColor`, `meta { slotKey,
description: { "<ratio>": brief }, silent? }`, `layers { "<id>": layer }`, `css?`, `gsap?`,
`sizeProps?`. Layer: `type`, `orderKey`, `data`, `style`, `meta`, optional `parentId`, `timing`,
`gsap`, `sizeProps`, `isHidden`, `subtitle`. Objects are strict: an unknown key is an error.

## Rules that decide quality

- **Ids** lowercase-dashed; layer ids unique across the package, never a scene id (`cover-headline`).
- **`orderKey`** is z-order among siblings: `a`, `b`, `c`…, later on top, unique, never ending in `0`.
- **Ground**: `common.style.backgroundColor` is the file background (else black); set each scene's too.
- **Fonts** by Fliki catalogue key (`montserrat`, `playFairDisplay`), never display name.
- **Other ratios** are `sizeProps`: `{ "9:16": { "style.top": 56.771, "style.font.size": 37 } }` —
  literal dotted keys, only differences, never the base ratio or `meta`; `null` removes a value.
- **Text boxes** are tall enough for the longest copy `meta.maxLength` allows; set `font.alignY`.
- **Media slots**: `fit: { "mode": "cover" }`, on a box that is exactly the rectangle seen.
- **Narration**: one `voiceover` per scene that has something to say, placeholder written from its
  copy; `meta.silent: true` on a scene silent by design.
- **B-roll**: `meta.broll: true` on a narrated scene's main picture — the only way videos from the
  template get cutaway shots. Hold tiles, logos, portraits.
- **Motion**: transforms, opacity, filter; `cw`/`ch` distances; finite repeats; nothing timed from
  the scene's end (narration resizes scenes) — use `transition` for the boundary.
- **Colours** as 6/8-digit hex, so palette switching reaches them.

## Safety rules (always refused)

- **Tags** (`data.tag`, and markup in `html`/`content`): `a div span p br hr strong b em i u s sub
  sup small mark h1–h6 ul ol li blockquote code pre section article header footer figure figcaption
  img picture`, table tags, and SVG shapes, text, gradients, masks, patterns and common filter
  primitives (exact list in [validation](references/validation.md#security)). No script, style, iframe,
  object, form, video or audio elements.
- **Attributes**: no `on*`, `srcdoc`, `dangerouslySetInnerHTML`, `ref`, `key`, `children`, `is`.
  `href`/`src`/`xlink:href`/`srcset`/`poster` only `asset:<file>` or `#id`. No `javascript:`,
  `vbscript:` or `data:` values. Class names `A–Z a–z 0–9 _ -`.
- **CSS** anywhere: `url()` only `asset:`, `#id`, Google Fonts or `data:image/…`; `@import` only
  Google Fonts; no `expression(`, `behavior`, `-moz-binding`, markup.
- **Tweens** never set `innerHTML`, `outerHTML`, `innerText`, `textContent`, `text`, `attr`, `on*`.
- **Fonts**: `builtin` keys, or `url` on `https://fonts.googleapis.com/css2?…`.
- **`description`**: plain markdown, no HTML.

## Example — validates clean

```json
{
  "format": "playback-v2-template", "revision": 1, "name": "Split Hero Brief",
  "settings": { "format": "video", "aspectRatio": "16:9" }, "aspectRatios": ["16:9", "9:16"],
  "fonts": { "montserrat": { "kind": "builtin", "weights": [700, 800] } },
  "palette": { "roles": [{ "key": "ground" }, { "key": "ink" }], "variants": [
    { "key": "paper", "colors": { "ground": "#F4F1EA", "ink": "#1B2433" } },
    { "key": "night", "colors": { "ground": "#11151C", "ink": "#E9E2D3" } } ] },
  "assets": { "cover.webp": { "kind": "image" }, "feature.webp": { "kind": "image" } },
  "common": { "id": "common", "layers": {}, "style": { "backgroundColor": "#F4F1EA" } },
  "scenes": [
    { "id": "cover", "name": "Cover — split hero", "customDuration": 5,
      "transition": { "kind": "fade", "duration": 0.5 }, "style": { "backgroundColor": "#F4F1EA" },
      "meta": { "slotKey": "cover", "description": { "16:9": "Claim left, photo band right.",
        "9:16": "Photo band on top, claim beneath." } },
      "layers": {
        "cover-photo": { "type": "media", "orderKey": "a", "data": { "mediaMyId": "asset:cover.webp" },
          "style": { "left": 55, "top": 0, "width": 45, "height": 100, "fit": { "mode": "cover" } },
          "meta": { "slotKey": "image", "description": "The cover picture, held." },
          "sizeProps": { "9:16": { "style.left": 0, "style.width": 100, "style.height": 52.083 } } },
        "cover-headline": { "type": "text", "orderKey": "b",
          "data": { "content": "Your big idea,<br>in one line" },
          "style": { "left": 6.25, "top": 30.556, "width": 42.708, "height": 27.778,
            "font": { "family": "montserrat", "size": 63, "weight": 800, "lineHeight": -12,
              "letterSpacing": -2, "color": "#1B2433", "align": "left", "alignY": "top" } },
          "meta": { "slotKey": "headline", "maxLength": 48, "description": "The claim. Two short lines." },
          "gsap": { "tweens": [{ "target": { "part": "line" }, "op": "from", "at": 0.3, "duration": 0.7,
            "vars": { "yPercent": 50, "opacity": 0 }, "ease": "power4.out", "stagger": 0.1 }] },
          "sizeProps": { "9:16": { "style.left": 7.407, "style.top": 56.771, "style.width": 85.185,
            "style.height": 29.167, "style.font.size": 37 } } },
        "cover-vo": { "type": "voiceover", "orderKey": "z", "style": {},
          "data": { "content": "Here is the one idea this video is about, said plainly." },
          "meta": { "slotKey": "script", "description": "The narration. One or two sentences." } } } },
    { "id": "feature", "name": "Feature — full-bleed photo", "customDuration": 6,
      "style": { "backgroundColor": "#1B2433" },
      "meta": { "slotKey": "feature", "description": { "16:9": "Point over footage, plate lower left.",
        "9:16": "Point over footage, plate near the bottom." } },
      "layers": {
        "feature-photo": { "type": "media", "orderKey": "a", "data": { "mediaMyId": "asset:feature.webp" },
          "style": { "fit": { "mode": "cover" } },
          "meta": { "slotKey": "broll", "broll": true, "description": "Footage for the narration." } },
        "feature-headline": { "type": "text", "orderKey": "b", "data": { "content": "One supporting point" },
          "style": { "left": 6.25, "top": 68.519, "width": 57.292, "height": 20.37,
            "fill": "#1B2433", "radius": 14.55,
            "font": { "family": "montserrat", "size": 46, "weight": 700, "lineHeight": -8,
              "color": "#F4F1EA", "padding": 7, "align": "left", "alignY": "middle" } },
          "meta": { "slotKey": "headline", "maxLength": 40 },
          "gsap": { "tweens": [{ "target": { "self": true }, "op": "from", "at": 0.2, "duration": 0.6,
            "vars": { "y": "4ch", "opacity": 0 }, "ease": "power3.out" }] },
          "sizeProps": { "9:16": { "style.left": 7.407, "style.top": 71.875, "style.width": 85.185,
            "style.height": 18.75, "style.radius": 8.89, "style.font.size": 25, "style.font.padding": 12 } } },
        "feature-vo": { "type": "voiceover", "orderKey": "z", "style": {},
          "data": { "content": "This is the point that supports it, with one concrete detail." },
          "meta": { "slotKey": "script" } } } }
  ]
}
```

Every number came from `template_units` (88px type at 1080 tall → `63`; 96px at 1920 tall → `37`).
Create with `assets: { "cover.webp": "<id>", "feature.webp": "<id>" }`.

## References

- [layers](references/layers.md) — layer types, slots, presenters, the common scene
- [style](references/style.md) — stored units, aspect ratios, fonts, palettes
- [motion](references/motion.md) — tweens, clocks, transitions, patterns
- [validation](references/validation.md) — every validation code and its fix

Without skills, `get_guide { topic: "templates", reference: "<name>" }` returns the same text.
