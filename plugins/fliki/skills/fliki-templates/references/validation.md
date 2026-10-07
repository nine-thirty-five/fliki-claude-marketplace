# Validation — every code and its fix

`template_validate` returns `errors` (they block `template_create`) and `warnings` (it saves, but
works worse), each `{ severity, scope, code, message }`. `scope` is `template.json`, `assets`,
`fonts`, `palette/<variant>`, `<scene>` or `<scene>/<layer>`, plus ` 9:16` when only that ratio is
affected. `(w)` marks a warning.

## Shape and structure

| code | fix |
|---|---|
| `schema.unrecognized_keys` | An unknown key (objects are strict): a typo, or a field on the wrong level. |
| `schema.invalid_type`, `invalid_value`, `too_small`, `too_big`, `invalid_union` | The field at `scope` has the wrong type, value, length or shape (`invalid_union`: `data` does not fit the layer `type`, or a malformed tween `target`). Schema errors come alone — fix them, then validate again. |
| `style.reservedKey` | `style.x`, `rotation`, `scale`, `transform`… Use `angle`, `flipX`/`flipY`, `width`/`height`, or a tween. |
| `package.revisionAhead` | Use `"revision": 1`. |
| `scene.duplicateId`, `layer.duplicateId` | Unique ids; layer ids across the package, never equal to a scene id. |
| `parent.missing`, `parent.crossScene`, `parent.notContainer`, `parent.self` | `parentId` must name a `container` in the same scene. |
| `order.invalid`, `order.duplicate` | `orderKey`: characters `0-9A-Za-z`, not ending in `0`, unique among siblings. |
| `order.mainPlane` | No layer may use `"V"` (the common scene's divider). |
| `singleton` | At most one `voiceover` and one `avatar` per scene. |
| `attachment.type`, `attachment.style` (w) | Only `media`, `voiceover`, `audio` carry a `subtitle`; give it a `style`. |
| `timing.window`, `timing.cuts`, `timing.mute`, `crop.at`, `crop.order` | `in` before `out`; each range's `start` before `end`; crop `at` within 0–1, ascending. |
| `sizeProps.branchValue` (w) | A patch writes a whole object over an existing one (`"style.font": {…}`); patch the leaves (`"style.font.size"`). |

## Files (`file_create`, `file_update`)

A file skips the template-only codes (slots, placeholders, palette, `voiceover.*`,
`asset.noDuration`, `asset.storedRef` for catalogue ids). An update is refused only for problems it
introduces — faults already in the file are reported by `file_read`, not held against an edit.

| code | fix |
|---|---|
| `ref.badId` | A `voiceId`, `voiceStyleId`, `speakers[].voiceId`, `avatarId`, `overlayId` or `fontId` that is not an id from a list tool. |
| `ref.notFound` | That voice, style, presenter, overlay or font does not exist. |
| `asset.unmapped`, `asset.badId` | Map every `asset:<name>` to an asset id in `assets`. |
| `asset.notFound`, `asset.kindMismatch`, `asset.notYours` | The id is missing, not the declared kind, or generated media of another account. |
| `asset.mixedUse` | One name used in two ref fields (`mediaMyId` and `mediaStockId`); give each its own name. |
| `asset.nameTaken` (update) | The name already stands for other media in this file; pick a new one. |
| `splice.invalid` (update) | `order` repeats an id, names one that exists nowhere, or leaves out a scene you sent. |
| `scene.pinnedNarration` (w) | `customDuration` on a narrated scene cuts a longer script off; remove it. |
| `template.tooManyScenes`, `template.sceneTooLong` | Template-only limits (80 scenes, 600 s a scene). |

## Units, geometry, aspect ratios

| code | fix |
|---|---|
| `style.outOfRange` | A value outside its scale (`font.size` 150, `opacity` 120). Convert with `template_units`. |
| `font.overflowsBox` (w) | Type taller than its box at that ratio — usually a size not converted for that canvas. |
| `style.degenerate` | `width`/`height` must be positive. |
| `style.dynamicHeight` | `height: -1` is not allowed here; give a real height. |
| `style.offCanvas` (w) | The box is entirely off canvas — typically converted against the wrong canvas. |
| `aspect.baseMissing` | `aspectRatios` must include `settings.aspectRatio`. |
| `sizeProps.activeAspectRatio` | Remove the patch for the base ratio. |
| `aspect.undeclared` | A patch for a ratio not in `aspectRatios`: declare it or remove the patch. |
| `sizeProps.badPath` | Patches may only address `data`, `style`, `timing`, `css`, `gsap`, `isHidden`, `customDuration` — never `meta`. |
| `aspect.noOverrides` (w) | A declared ratio nothing patches. Lay it out, or drop it from `aspectRatios`. |

## Assets

| code | fix |
|---|---|
| `asset.undeclared` | An `asset:<file>` is used but not declared in `assets`. |
| `asset.unused` (w) | Declared but never referenced: remove it. |
| `asset.noDuration` | Video and audio need `duration` (seconds) — copy it from the asset (`list_assets`). |
| `asset.storedRef` | A raw id in `mediaMyId`, `avatarId`, `voiceId`, `overlayId`… Use `asset:<file>` or delete it. |
| `asset.badRef` | Malformed reference — `asset:` plus a name of letters, digits, `. _ - /`. |
| `asset.legacyPath` | The text `assets/` appears somewhere, even in copy. Use `asset:<file>`. |
| `asset.unmapped` (create) | `template_create` got no asset id for that file. Add it to `assets`. |
| `asset.kindMismatch` (create) | The asset does not exist or is not the declared `kind` (image/video/audio). An id that is not the user's fails the call. |

## Fonts

| code | fix |
|---|---|
| `font.unlisted` | A layer uses a family missing from `fonts`. Add it. |
| `font.unknownBuiltin` | Not a catalogue key: `playFairDisplay`, not `Playfair Display` (style.md). |
| `font.storedRef` | `kind: "custom"` is refused. Use `builtin` or a Google Fonts `url`. |
| `font.hrefNotStylesheet` | A `url` href pointing at a font file. Use a `https://fonts.googleapis.com/css2?…` stylesheet. |
| `font.bundledFace` | An `@font-face` loading `asset:`. Remove it; declare the family in `fonts`. |
| `font.inInlineStyle`, `font.inCss` (w) | `font-family` in css or `inlineStyle` (an error on a `text` layer). Use `style.font.family`. |
| `font.unused` (w) | A declared family no layer uses. |

## Motion and transitions

| code | fix |
|---|---|
| `gsap.reflowTween` | A tween animates layout (`left`, `width`, `fontSize`, `padding`…). Use `x`/`y`/`scale`/`opacity`. |
| `gsap.infiniteRepeat` | `repeat: -1`. Use a finite count with `yoyo`. |
| `gsap.targetMissing` | `{ layerId }` names no layer in this scene. |
| `gsap.labelMissing` | `at: "name+=…"` uses a label not in `labels`. |
| `gsap.unknownPart` (w) | That layer type has no such part (see motion.md). |
| `gsap.overrun` (w) | The tween ends after `customDuration`. Shorten it or start it earlier. |
| `transition.unknownKind` | Use `none blur clock fade flip slide wipe zoom`. |
| `transition.unknownSfx`, `unknownEase` (w) | Keys in motion.md. |
| `transition.trailing` (w) | Remove the last scene's transition. |

## Narration, slots, presenters

| code | fix |
|---|---|
| `voiceover.templateMissing` | A `video`/`audio` template needs at least one `voiceover` layer. |
| `voiceover.sceneMissing` (w) | A scene with real copy has no voiceover: add one, or set scene `meta.silent: true`. |
| `voiceover.silentWithScript` | `meta.silent` and a voiceover in one scene — remove one. |
| `template.emptyScript` (w) | Give the voiceover a placeholder script. |
| `layer.untyped` | A `generic`/`container` that is just one `<img>`, `<svg>` or `<audio>`: make it `media`, `vector` or `audio`. |
| `slot.fitStretches` (w) | A fillable media slot whose `fit` is absent or `fill`. Use `cover` (`contain` for a logo). |
| `slot.boxOverflowsFrame` (w) | A slot larger than the container clipping it. Give it the frame's box. |
| `slot.duplicate` | Two layers in one scene share a `slotKey`. |
| `scene.noFillableSlot` (w) | Nothing in the scene has a `slotKey`. |
| `template.noSlotKey` (w) | No scene has `meta.slotKey`; it can never be picked as a layout. |
| `template.longCopy` (w) | Placeholder longer than 120 chars (or `maxLength`). Shorten it. |
| `avatar.noPlaceholder` (w) | Give the avatar a placeholder `data.mediaMyId`. |
| `avatar.background` (w) | Set `data.background: "remove"`. |
| `avatar.layout` (w) | Circle layout needs a square-in-px box, `radius: 100`, `layoutFrame`, no crop. |
| `avatar.mediaFlag` (w), `avatar.sourceWindow` (w) | Remove `broll`, `reserved`, `removeBackground`… and `timing.cuts`/`mute`. |

## Paint and palette

| code | fix |
|---|---|
| `background.fileMissing` (w) | Set `common.style.backgroundColor`. |
| `background.sceneMissing` (w) | Nothing paints the scene: set its `style.backgroundColor`. |
| `plane.frontCovers` (w) | An opaque full-bleed common layer above `"V"` hides every scene. Key it below `V`. |
| `css.notPainted` (w) | A layer's own `css` is ignored. Move the rules to the scene `css`. |
| `palette.missing` (w), `palette.singleVariant` (w) | Add a palette with 2–4 variants. |
| `palette.roleMissing`, `unknownRole`, `duplicateRole`, `duplicateVariant`, `unknownActive` | Every variant states exactly the declared roles; keys unique; `active` names a variant. |
| `palette.duplicateColor` | Two roles share a value in one variant. Change one, or merge them. |
| `palette.notHex`, `palette.redundantAlpha` (w) | Role values are `#RRGGBB` or `#RRGGBBAA`; drop a trailing `FF`. |
| `palette.roleUnused` (w) | A role's colour appears nowhere in the package's paint. |
| `palette.coverage` (w) | Painted colours no role covers. Add roles, or make them tints of one. |
| `palette.nonHexOccurrence` (w) | `rgb()`, `hsl()` or 3-digit hex in css/data. Write 6/8-digit hex. |

## Security

Every `security.*` finding is an error, and nothing is stripped for you: rewrite the source.

| code | refused |
|---|---|
| `security.tag` | An element outside the allowlist below (in `data.tag` or any markup). |
| `security.attrName` | `on*` handlers, `srcdoc`, `dangerouslySetInnerHTML`, `ref`, `key`, `children`, `is`, or a name outside `[A-Za-z_][A-Za-z0-9_.:-]*`. |
| `security.attrUrl` | `href`, `xlink:href`, `src`, `srcset`, `poster`, `data`, `background`, `action`, `formaction` pointing anywhere but `asset:<file>` or `#id`. |
| `security.attrScheme` | Any attribute value starting `javascript:`, `vbscript:` or `data:`. |
| `security.attrValue`, `security.shape` | Attribute values must be strings; `attrs`, `inlineStyle` and tween `vars` must be objects. |
| `security.cssUrl` | `url()` other than `asset:`, `#id`, Google Fonts, or `data:image/…`. |
| `security.cssImport` | `@import` other than Google Fonts. |
| `security.cssScript` | `expression(`, `javascript:`, `vbscript:`, `behavior:`, `-moz-binding`. |
| `security.cssMarkup` | `<style` or `<script` inside css. |
| `security.tweenProperty` | Tween keys `innerHTML`, `outerHTML`, `innerText`, `textContent`, `text`, `attr`, `on*`. |
| `security.className` | Class names other than letters, digits, `_`, `-`. |
| `security.fontUrl` | A `url` font not on Google Fonts. |

CSS is checked everywhere it can appear (scene `css`, `inlineStyle`, `style` attributes and
values, tween values, palette colours), after decoding comments and escapes; markup in
`data.html`, `data.content`, `data.text` and `description`.

**Allowed elements**, and nothing else:

- Text and structure: `a div span p br hr strong b em i u s sub sup small mark h1 h2 h3 h4 h5 h6
  ul ol li blockquote code pre section article header footer figure figcaption img picture`
- Tables: `table thead tbody tfoot tr td th caption colgroup col`
- SVG: `svg g path rect circle ellipse line polyline polygon defs symbol use text tspan textPath
  title desc linearGradient radialGradient stop clipPath mask pattern filter image`
- SVG filters: `feBlend feColorMatrix feComponentTransfer feComposite feDropShadow feFlood feFuncA
  feFuncB feFuncG feFuncR feGaussianBlur feMerge feMergeNode feMorphology feOffset feTurbulence
  feDisplacementMap`

Not allowed, among others: `script`, `style`, `iframe`, `object`, `embed`, `video`, `audio`,
`source`, `form`, `input`, `button`, `link`, `meta`, `foreignObject`, SVG `animate`/`set`.
