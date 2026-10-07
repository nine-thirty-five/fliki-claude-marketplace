---
name: fliki
description: Start here for any Fliki request. It reads what the user actually wants, picks the path that fits (one generation, a video authored scene by scene — the default for composed videos — a Fliki workflow, a template, or an edit), asks only the questions that change the result, and routes to the right guide. Fliki's MCP tools create images, short video clips, music, speech, complete narrated videos, audio files, designs (thumbnails, ad creatives, carousels, presentations), authored files and reusable templates, and find the user's Fliki files, exports, assets, voices, characters, brand kits and templates. Use whenever the user asks to make, generate, edit, render or export media with Fliki, mentions their Fliki account, or wants something Fliki's tools can produce.
---

# Fliki

Fliki's tools act on the user's own Fliki account. Everything they create lands in their Fliki
library and spends their Fliki credits.

## Read the intent first

Several paths can make the same request, and the wrong one spends credits on a guess. Before any
generating call, read five things from what the user said and attached:

1. **Deliverable.** One of:
   - one asset (an image, a clip, a track, a line of speech);
   - a composed piece (a narrated video, an audio file, a set of designs);
   - a reusable layout;
   - a change to a file they already have.
2. **Control.** Do they name specifics (exact words, shots, order, brand, timing), or leave the
   choices to you ("a video about X")?
3. **Source.** What they start from:
   - an idea or a script;
   - an article or site link;
   - product photos, or their own footage or images;
   - a music track;
   - an existing Fliki file.
4. **Craft.** Must it land something — a story, an ad's beats, a documentary's facts, an edit's
   rhythm? Or is clear and on topic enough?
5. **Constraints.** Platform and aspect ratio, length, language, voice, brand kit, presenter,
   budget, and how soon they need it.

## Pick the path

| Path | Best when | Trade-off | Guide |
|---|---|---|---|
| **Playground**: `generate_image`, `generate_video`, `generate_music`, `generate_speech` | One asset, or variations of one | Nothing is composed: no scenes, narration or captions | fliki-playground |
| **Authored file**: storyboard → `file_create` | **The default for any composed video.** Explainers, promos and ads, stories, documentaries, topic and social videos, a short script or page turned into video, edits of the user's footage | You write every scene, layout, line and GSAP tween, and source every asset. Suits 3–15 scenes | fliki-files |
| **Workflow**: `list_workflows` → `start_workflow` | Only what an authored file can't do well (see below) | Fliki plans the scenes and picks the visuals. You steer through the inputs only | fliki-workflows |
| **Template**: `template_create` | A layout reused for many videos | Makes a layout, not a video | fliki-templates |
| **Edit**: `file_read` → `file_update` | A change to a file they already have | — | fliki-files |

### Authored file or workflow

For a composed video, author it. A storyboard plus a playback-v2 file gives you exact copy, layout,
typography, GSAP motion, common-scene continuity, the user's media and generated shots exactly
where they belong. A workflow re-plans from inputs and lands near the brief, not on it.

Use a workflow only when one of these holds:
- **A music video**: a song with picture cut to its beat → `music-video`.
- **Heavy AI video**: the piece is mostly generated footage, such as a cinematic or character-driven
  story told in many AI clips, where Fliki's shot planning and batch clip generation do the work
  → `explainer-video` or `script-to-video` with `visuals: ai-video`. A few AI shots inside an
  otherwise designed video stay in the authored file (generate them in the playground first).
- **Long form**: more than about 15 scenes, a long script or document, or an audiobook →
  `script-to-video`, `url-to-video`, `audiobook`.
- **Audio only or a design**: `script-to-audio`, `url-to-audio`, `music-audio`,
  `directed-voiceover`, `thumbnail`, `ad-creative`, `social-post`, `presentation`.
- **The user asks** for a named workflow, or for the fastest result with Fliki choosing the visuals.

Paths combine, and the combination is often the best answer:
- **Playground → file.** Generate the exact shots (consistent characters, product close-ups, AI
  clips), wait for each to finish, then compose them in a file with narration, captions, music and
  motion. This is the default for stories, video ads and short documentaries.
- **Workflow → edit.** A workflow does the bulk, then `file_update` fixes the scenes that must be
  exact.
- **Template → workflow.** For a recurring series in one look: build the template once, then run
  `script-to-video` or `url-to-video` with its `templateId` for each episode.

| The user wants | Recommend |
|---|---|
| A story (short film, kids' story, fable) | Storyboard → playground shots with consistent characters → authored file. Mostly AI footage and long: `explainer-video` with `visuals: ai-video` |
| A video ad or promo | Storyboard → authored file on the product's photos, footage, website look or generated shots |
| Static ads, a thumbnail, a carousel, a deck | `ad-creative`, `thumbnail`, `social-post`, `presentation` |
| A documentary | Storyboard with sourced facts → authored file. Long: `script-to-video` over stock |
| An explainer or topic video | Storyboard → authored file |
| A video of an article or page | Storyboard from the page → authored file. A long article: `url-to-video` |
| A narrated video of their script | Storyboard on their script → authored file. A long script: `script-to-video` |
| A short social clip | Authored file at 9:16. A single `generate_video` if it is one shot |
| Motion graphics, kinetic type, a product reveal | Authored file: layouts and GSAP are yours to write |
| A music video | `music-video`. For a concept piece: storyboard → `music-audio` + playground shots → authored file |
| A presenter-led video | Authored file with an `avatar` layer per scene, lip-synced by `file_generate` |
| A podcast or several voices | `directed-voiceover` |
| Narration only, or an audiobook | `script-to-audio`, `url-to-audio`, `audiobook` |
| Their own footage edited | `list_assets` (or `upload_assets`) → `transcribe_asset` → storyboard → authored file |
| Changes to one of their videos | `file_read` → `file_update` |

## Ask what the request leaves open

If the five reads leave the path or the result open, ask before creating anything:

- **Ask one round of at most four questions.** Give each 2–4 concrete options and mark your
  recommended default, so "defaults" is a complete answer. Use the app's multiple-choice UI if it
  has one; otherwise ask as a short numbered list.
- **Offer real options, pulled from the tools, never from memory:**
  - 2–4 voices from `list_voices`, by name, accent and style;
  - templates from `list_templates`, by name and category;
  - the user's brand kits from `list_brand_kits` (their own and their team's);
  - looks from `list_presets`: caption styles for an authored file; thumbnail styles, carousel,
    deck and ad looks for design workflows (animation presets only when the user asks for one);
  - presenters from `list_characters { type: avatar }`;
  - models from `list_models`, with what they cost.
- **Ask only what changes the result.** Skip anything a default settles:
  - the aspect ratio follows the platform, and is 16:9 when none is named;
  - the voice follows the language.

  Skip anything the path ignores.
- **Ask plainly for facts only the user knows,** such as the business name, the offer or the call
  to action. Ask them alongside the option questions.
- **Close with a recommendation:** the path, its trade-off in one line, and a rough cost. For a
  generation, the cost is a number from `list_models`. For a workflow, name what drives it: AI
  video visuals cost the most. For example: "I'll storyboard it, you approve every scene, then I
  build it scene by scene with your copy, motion and visuals; narration costs about N credits."
  Offer a workflow alongside only when it fits the list above.
- **If the user says "just make it",** skip the questions and the storyboard, and take your
  defaults. The one confirmation left is the cost check below: state your defaults and the cost in
  one line, then start on their yes.

## Storyboard before composed content

A composed video or audio piece gets a storyboard (fliki-storyboard), approved before anything is
generated. The user sees the script, the look and every scene while changes are still free.

When a workflow runs on complete input, such as a long script or an article, a short plan card
replaces the storyboard.

Skip both for a single asset, a design workflow, or a small edit. Skip them too when the user
says to go straight to creation.

## Rules that apply everywhere

- **Never invent an id.** Every id comes from a tool:
  - voices from `list_voices`;
  - templates from `list_templates`;
  - brand kits from `list_brand_kits`;
  - models from `list_models`;
  - assets from `list_assets`, `upload_assets`, `import_assets` or a generate tool.
- **Check cost first.** `list_models` shows credits per image or per second, and `get_account`
  shows what is left. Before a video clip, a long track or a workflow, tell the user roughly what it
  costs and get a yes. Never loop generations to "try again" without asking.
- **Long jobs return at once.** They hand back something to follow:
  - `generate_video`, `generate_music` and some image models return a `handle`: follow it with
    `get_generation`;
  - `file_generate` (narration and presenter lip-sync of an authored file) holds with `wait: true`;
  - `start_workflow` and `export_file`: follow them with `get_file`.

  Never build on or export something still generating: wait for every clip, track and presenter
  to finish first.

  Where long tool calls run in the background (Claude Code), call them with `wait: true`: the call
  holds until the job finishes, up to about 9 minutes, then call again. Otherwise poll
  `get_generation` every 15–30 seconds and `get_file` every 30–60 seconds. Tell the user it is
  running rather than waiting silently. Stop after about 20 minutes and tell them they can check
  later.
- **Bring outside media in first.** Both routes take several files per call and return asset ids.
  - First look for the file in the library: `list_assets { query: "<file name>" }`. A match with
    the same `size` is the same file: use its `id` and skip the upload. A `transcribed` match
    already has its words, so `transcribe_asset` returns them at once.
  - A file the user attached or has on disk takes three steps, and needs a shell or code tool that
    can reach the url, because the bytes go straight to storage:
    1. `upload_assets { files: [{ name }] }`.
    2. PUT each file to its `url` with its `headers`:
       `curl -sS -X PUT -H "content-type: …" -H "x-goog-content-length-range: …" --upload-file <path> "<url>"`.
    3. `finish_uploads { uploadIds }`.
  - Media on the web: `import_assets { items: [{ url }] }`. Public https links only.
  - With neither route: ask the user to upload the files in Fliki, then find them with
    `list_assets`.
- **Show results as links.** Share `url`, `downloadUrl` or `editorUrl` so the user can open the
  result in Fliki to preview, tweak or download it.
- **Errors are for the user.** A tool error is written to be read:
  - Relay it. If it says what is wrong (an unknown model, a locked plan feature, a missing field),
    fix the input.
  - A rate limit says how long to wait. Wait that long before retrying, and never loop on
    generation tools.
  - At most two workflows run at once per account.

## Account

Fliki's tools need a Fliki Premium or Enterprise plan. If a call says the plan does not allow
something, suggest upgrading at fliki.ai rather than retrying. Connected apps are managed in Fliki
under **Automation → AI apps (MCP)**.

Other guides: fliki-storyboard, fliki-playground, fliki-workflows, fliki-files, fliki-templates. In
apps without skills, the `get_guide` tool returns the same text.
