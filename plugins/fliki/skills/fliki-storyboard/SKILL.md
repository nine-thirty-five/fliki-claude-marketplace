---
name: fliki-storyboard
description: Plan a Fliki video or audio piece before creating it. Work out what the user wants, then write a storyboard covering the logline, mood and style, cast and voices, the full script, every scene and shot, music and captions, and the assets each scene needs with their cost. Show it as an artifact the user reviews and approves, then build exactly that as an authored file, a workflow or playground shots. Use before a story, ad or promo, documentary, explainer, music video, edit of the user's footage or any multi-scene piece; when the brief is thin; or when the user asks for a storyboard, script, shot list, treatment or plan.
---

# Fliki storyboard

A storyboard puts the whole piece on one page before anything is generated: what it says, how it
looks and sounds, and what every scene shows. It costs nothing, and it is where a vague brief
becomes the video the user meant. Once it is approved, build to it.

## When

- **Always** write one for a composed video or audio piece. That includes:
  - a story, an ad or promo, a documentary, an explainer, or a music video;
  - a podcast, or an edit of the user's footage;
  - any multi-scene authored file.
- **Use a plan card instead** (see section 2) when a workflow will run on complete input, such as
  a finished script or an article. The user sees the inputs and what Fliki will decide.
- **Skip it** for a single asset, a design workflow, a small edit, or when the user says to go
  straight to creation. Take your defaults and say which you took.

## 1. Gather

Start from the intent reading in the fliki guide. If the path or the result is still open, ask its
one round of questions. Then collect what the storyboard names:

- **A real id for every choice:**
  - voices from `list_voices`;
  - a template from `list_templates`;
  - a brand kit from `list_brand_kits`;
  - a presenter from `list_characters { type: avatar }`;
  - models from `list_models`.
- **The user's media, brought in first**: reuse what `list_assets` already holds, otherwise
  `upload_assets` → PUT → `finish_uploads`, or `import_assets`, so scenes can name it. Their footage and photos shape the scenes. When you
  can't see a clip, ask what it shows.
- **For anything factual** (documentaries, news, claims in an ad), the facts, each with its
  source. Leave out what you could not source.
- **The look they already have**: a brand kit's colours, fonts and logo, or the palette and type of
  a site they share.

## 2. Write it

| Section | Holds |
|---|---|
| **Overview** | Title and a one-sentence logline. Audience and platform. The one thing the viewer should feel, know or do. Format, aspect ratio, length, language. The path and why: an authored file unless the piece is one the fliki guide sends to a workflow |
| **Mood & style** | The visual system every prompt and layout follows: medium (live action, 3D, illustration, motion graphics), palette as hex, heading and body type, lighting and texture, how things move, references |
| **Cast & world** | For each recurring character, product and place: its look in one line, and the reference that keeps it consistent (a user photo or a generated reference image). The voice for each speaker, by name and style. A presenter, if any |
| **Script** | Every word spoken, labelled by speaker where there are several. It sets the length: about 150 words a minute |
| **Scenes** | One row or card per scene, as below |
| **Sound** | Music (genre, mood, tempo, where it lifts or drops), sound effects, and caption style or none |
| **Assets & cost** | Each generation with its model, quality, seconds or count, and credits. What comes from the user or from stock. The total against `get_account`, and how long it will take |
| **Open points** | Every default you chose and every question left, so the user can change them in one reply |

Each scene gives:
- **Number, time and length.** Narration decides a narrated scene's length. Set silent scenes
  (stings, end cards) in seconds.
- **Purpose.** Its one job: hook, problem, reveal, proof, turn, or call to action.
- **Visual.** A sequence of shots, each with one framing, one subject, one action and one camera
  move: "close-up, flour dusts the counter as hands fold dough, slow push-in". Cut on the
  narration's beats rather than holding one picture.
- **On-screen text.** The exact words.
- **Narration or dialogue.** The scene's lines of the script.
- **Sound.** Music cue and effects.
- **Transition** into the next scene: a stock `transition`, or a cross-scene element staged in
  the common scene (a shape that wipes across the cut, a title that stays while the scene changes
  under it) when a stock one can't do it.
- **Source** for each shot: generated (with model and duration), the user's asset, stock, or a
  template slot.

**Shape the story before the scenes.**
- Hook the viewer in the first 2–3 seconds.
- One idea per scene.
- End by paying off the hook, or by asking for one action.
- Scene length: 3–6 seconds for short social pieces, 5–12 for explainers and documentaries.

**Plan card** (a workflow on complete input). It lists:
- the workflow;
- every input it will get: voice, template, brand kit, aspect, visuals, captions, length;
- what Fliki will decide on its own;
- cost and time.

## 3. Show frames (optional)

Pictures settle the look faster than words. Offer them, with their cost, before making any:

1. **A style frame.** Make one from the mood and style with `generate_image`, at the piece's
   aspect ratio. Get it approved before anything else.
2. **Key frames** for every scene, or just the key ones. Pass the style frame and the cast
   references as `referenceAssetIds`.

These frames are not throwaway. Approved frames become the build: each clip's
`firstFrameAssetId`, or the scene's picture.

## 4. Present it

Put the storyboard where the user can read and review it, not in the chat stream:

- **In apps with artifacts (Claude)**, make one HTML artifact.
  - Style it in the storyboard's own palette and type, so the page previews the look.
  - Lead with a summary bar: length, aspect, path, credits.
  - Then, in order:
    1. the overview;
    2. mood and style, with palette swatches;
    3. the cast and their voices;
    4. one card per scene: its frame (or a described placeholder), time, visual, on-screen text
       and narration.
  - Show generated frames inline from their `url`. Name templates, characters and voices; the
    user can preview them in Fliki.
- **In coding agents without artifacts**, write `storyboard.html` (or `storyboard.md`) in the
  working folder and give its path.
- **Elsewhere**, send a formatted message: the overview, then a scene table.
- **A plan card** is always a short message.

In the chat, say in two or three lines what the storyboard proposes and what you need from the
user.

## 5. Review

- **Revise in place.** Change the same artifact, bump its version (v2, v3), and list what changed.
- **Wait for approval.** Go ahead only on a clear approval of the storyboard or plan card. Spend
  nothing before it, except on frames the user agreed to.
- **Treat the approved storyboard as the spec.** If the build can't match it — a model can't do a
  shot, a scene runs long — say so and propose the fix. Don't change it silently.

## 6. Build to it

**Authored file** (fliki-files guide). Generate each asset on the list, since its cost is
approved, then write one package scene per storyboard scene:

| Storyboard | Package |
|---|---|
| Narration | The scene's `voiceover` `data.content` |
| On-screen text | Text layers |
| Shots | Media layers on `asset:` names: cut in sequence, or placed with `data.generate.reference` on the phrase each one illustrates |
| Music | An `audio` layer in `common` |
| Voice | `voiceId` |
| Transitions | `transition` |
| Mood & style | Colours, fonts and motion |
| Cross-scene elements, persistent frame, backdrop | Layers in `common`, above or below `"V"` |
| Presenter | An `avatar` layer in the scene |

Generate every clip, image and track first and wait for each (`get_generation` with
`wait: true`) before writing it into the package. Then `file_create`, share `editorUrl`, and run
`file_generate { wait: true }` so the narration and presenters exist before you place
timing-dependent layers. Export when the user wants the file.

**Workflow** (fliki-workflows guide):
- `script-to-video`:
  - the script goes in `script`; leave `targetMinutes` off to keep it as written;
  - mood, style and pacing go in `instructions` (up to 2000 characters);
  - visuals, voice, template, brand kit and aspect go in their own inputs.
- `explainer-video`, `motion-video`, `music-video`:
  - condense the storyboard into `brief`: the logline, the style, then the beats scene by scene;
  - `brief` is up to 5000 characters, or 3000 for `music-video`;
  - Fliki re-plans from the brief, so the result follows it but not every shot.
- When the file is `ready`, compare it with the storyboard and offer `file_update` for the scenes
  that miss.

**Playground shots** (fliki-playground guide). Make each shot with one `generate_video`:
- the shot's line is the prompt, with the mood and style added to every prompt;
- its key frame is the `firstFrameAssetId`;
- on models whose `referenceImages` is above 0, pass the cast references as `referenceAssetIds`.

Deliver the clips as they are, or compose them in a file.
