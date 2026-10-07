---
name: fliki-playground
description: Generate single pieces of media with Fliki's models — an image, a short AI video clip, a music track or a voiceover line — and get the result back. Use when the user asks Fliki for "an image of…", "a clip of…", "a song/track/jingle", "read this in a voice", image-to-video, or wants to choose between Fliki's image, video or music models.
---

# Fliki playground

The playground makes one asset per call. Several shots that together form one piece, such as an
ad, a story or a sequence, are storyboarded first (fliki-storyboard) and then composed in a file
(fliki-files).

## 1. Choose a model

Call `list_models { kind }` for `image`, `video` or `music` and pick by what the job needs:

- Skip models with `locked: true`; they need a higher plan.
- **Image**: `creditsPerImage`; `referenceImages` > 0 if the user gave reference pictures.
- **Video**: `textToVideo` / `imageToVideo`, `firstAndLastFrame`, `nativeAudio` (the clip carries
  its own sound), `durations`, `aspectRatios`, and `qualities[]` with `creditsPerSecond`.
  Cost ≈ `creditsPerSecond × duration`.
- **Music**: `durations`, `lyrics` support, `creditsPerSecond` / `creditsPerGeneration`.

Omit `model` to use Fliki's default. Pass the model `id` exactly as listed.

## 2. Generate

**Image** — `generate_image { prompt, model?, aspectRatio: 1:1 | 16:9 | 9:16, referenceAssetIds? }`.
Returns `status: succeeded` with the asset, or `status: pending` with a `handle`.

**Video clip** — `generate_video { prompt, model?, quality?, duration?, aspectRatio?, firstFrameAssetId?, lastFrameAssetId?, referenceAssetIds? }`.
Always returns a `handle`. Use a `quality` and `duration` the model lists.
- Image-to-video: generate or import the still first, then pass its id as `firstFrameAssetId`.
- A clip that ends on a set frame needs a model with `firstAndLastFrame`.

**Music** — `generate_music { prompt, lyrics?, model?, duration? }`. Returns a `handle`
(a track takes a minute or two).

**Speech** — `generate_speech { voiceId, styleId?, text }`.
- Find the voice with `list_voices { language?, gender?, query? }`. Use `list_languages` for
  non-English, and a style id from the voice's `styles` when there is one.
- Returns a temporary `audioUrl`; tell the user to download it. It is not saved to their library.

**Poll** — `get_generation { handle }` every 15–30 seconds until `succeeded` or `failed`, or once
with `wait: true` where long tool calls run in the background (Claude Code). Video usually takes
1–8 minutes.

## 3. Prompt well

- **Image**: subject, setting, composition, lighting, style, colour palette. Name the aspect in
  the scene ("wide establishing shot").
- **Video**: one shot, one camera move, one action. Say what moves and how ("slow push-in as
  steam rises from the cup"). Avoid scene changes inside one clip.
- **Music**: genre, mood, tempo, instruments, vocal style. Put lyrics in `lyrics`, not the prompt.

## 4. Use what already exists

- The user's own files → `upload_assets`, PUT each, then `finish_uploads`; web media →
  `import_assets { items: [{ url }] }`. Use the returned asset `id`s.
- The user's library → `list_assets { type, source, query }`.

Generated images, clips and tracks are saved to the user's Fliki assets and can be reused as
references, in workflows and in templates.
