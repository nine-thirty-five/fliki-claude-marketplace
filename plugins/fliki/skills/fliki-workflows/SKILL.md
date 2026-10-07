---
name: fliki-workflows
description: Produce a complete Fliki piece from a few inputs where Fliki plans it — a music video, a video made mostly of AI-generated footage, a long script or article to video, narrated audio, a song, multi-voice voiceover, an audiobook, a thumbnail, ad creatives, a social carousel or a presentation — then export it for download. Use when the fliki guide routes to a workflow, when the user names one, or when they ask to export or download a Fliki file. Most composed videos are authored scene by scene instead (fliki-files).
---

# Fliki workflows

Workflows are the exception path. Use one for a music video, a piece made mostly of AI-generated
footage, long form (more than about 15 scenes), audio-only work, a design, or when the user names
a workflow or wants Fliki to choose the visuals. Every other composed video is authored scene by
scene from a storyboard (fliki-files), because that hits the brief exactly.

Choose a workflow only after the fliki guide's intent reading has settled the path. Then, unless
the user said to go straight to creation, get the user's approval first (fliki-storyboard):
- on a storyboard for a composed video or audio piece;
- on a plan card when the input is already complete, such as a finished script or an article.

The approved version becomes the inputs.

## 1. Pick the workflow

Call `list_workflows`. Each entry has a `key`, what it `produces`, and the JSON schema of its
`inputs`.

| Goal | key |
|---|---|
| Narrate a script over stock footage, AI images or AI clips | `script-to-video` |
| Turn a blog post or page into a video | `url-to-video` |
| Fully generated explainer from a brief | `explainer-video` |
| Short, design-led motion graphics (optionally from a website) | `motion-video` |
| Song plus a video cut to its beat | `music-video` |
| Narrated audio from a script or article | `script-to-audio`, `url-to-audio` |
| A song | `music-audio` |
| Several voices performing a script ("HOST: …" lines) | `directed-voiceover` |
| Long-form narrated book | `audiobook` |
| Thumbnail, ad creatives, carousel, deck | `thumbnail`, `ad-creative`, `social-post`, `presentation` |

## 2. Gather inputs

Ask only for what the schema needs and the user has not said.

- **Voice**: optional. Fliki picks one for the language. If the user cares, `list_voices` and
  pass a `voiceId` that is not `locked`.
- **Template** (script/url video): `list_templates { source: fliki | mine }` → `templateId`.
- **Brand kit**: `list_brand_kits` → `brandKitId`.
- **Design looks**: `list_presets` lists them; show 3–4 that suit the topic, with previews where
  given, and offer Auto (omit) as the default:
  - `thumbnail`: `presetId` from `{ kind: "thumbnail" }` (not with `referenceAssetIds`);
  - `social-post`: `preset` + optional `palette` from `{ kind: "carousel" }`;
  - `presentation`: `preset` + optional `palette` from `{ kind: "deck" }`;
  - `ad-creative`: `preset` from `{ kind: "ad-creative" }`, picked by `fits` for the product.
- **Aspect ratio**: `16:9` for YouTube and slides, `9:16` for Reels/TikTok/Shorts, `1:1` for feeds.
- **Product photos** (ad creatives) or **reference images** (thumbnails): asset ids from
  `list_assets`, `upload_assets` or `import_assets`.
- **Links** (url workflows): public https only.

Workflows spend credits, and AI-video visuals cost the most. Confirm before starting, especially
for long durations or `visuals: ai-video`.

## 3. Start, then poll

`start_workflow { workflow, inputs }` returns a `fileId` and an `editorUrl` right away while
Fliki generates in the background.

Poll `get_file { id }` every 30–60 seconds, or call it with `wait: true` where long tool calls run
in the background (Claude Code) — it returns when the status changes, or after ~9 minutes:
- `generating` — keep waiting. Typical runs: short videos 3–10 minutes, explainers and music
  videos 10–25 minutes, designs 1–5 minutes.
- `ready` — done.
- `failed` — relay the error and offer to retry.
- `not_started` — opening `editorUrl` in Fliki resumes it.

Explainer, motion and music videos run one at a time per account. If a start says one is already
running, wait for it to finish.

## 4. Export and deliver

When `get_file` says `ready`:

1. Call `export_file { fileId, resolution?, format?, aspectRatio? }`.
   - Video: `mp4` / `mov`. Audio: `mp3` / `wav`. Design: `jpg` / `png` / `webp` / `pdf` / `pptx`.
2. Poll `get_file` until `export.downloadUrl` appears.
3. Share the link, plus `editorUrl` for edits in Fliki.

`list_files` and `list_exports` find earlier work. To change specific scenes of a finished file —
copy, timing, media, motion — use `file_read` and `file_update` (fliki-files guide).
