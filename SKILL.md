---
name: memegen-video-cover
description: >-
  Produce a high-quality AI video-meme cover — a parody music video where a
  chosen artist (e.g. Viktor Tsoi) performs reworked lyrics over a soundalike of
  a famous song, with a cinematic regenerated video. Use this when the user wants
  to make a "meme cover": new lyrics + cloned artist voice + AI video with lipsync.
  Covers the full pipeline: de-plagiarized meter-locked lyrics → Suno Add-Vocals
  over the REAL separated minus → RVC artist voice → Higgsfield image+video
  (Nano Banana Pro + Veo 3.1) → fal Sync 2.0 Pro lipsync → whisper-aligned ffmpeg
  assembly → composio YouTube publish. The working star lives at work/memegen.
---

# memegen-video-cover

The hard-won, end-to-end recipe for an AI **video-meme cover**. First real run shipped to
YouTube: Kino «Восьмиклассница» → «Онлифанщица» — Tsoi voice on the real Kino minus, 3:03
Soviet-noir/Blade-Runner video, 38 lyric-aligned cuts, 9 lipsync slots.

Project lives in the **memegen star**: `…/obsidian/work/memegen/`. Per-meme artifacts in
`out_memes/<slug>/` (audio takes, `video/`, `video/clips/`, `brief.json`, `lyrics.txt`).

---

## The flow (high level)

```
LYRICS  ─ meter-locked to the original + de-plagiarized (pass Suno copyright filter)
AUDIO   ─ separate REAL minus (BS-Roformer) → Suno "Add Vocals" over it → RVC artist voice
VIDEO   ─ storyboard whole song → SEED per shot → CLIP per shot (raw, numbered) → approve each
ASSEMBLE─ transcribe vocal (whisper) → cut shots ONTO their lyric lines → lipsync singers → concat
PUBLISH ─ composio → YouTube (private upload + thumbnail) → user flips Public in Studio
```

This is a **direct dynamic workflow with the user — NO Archon.** Generate in background
batches; present every artifact as a frame-strip / contact-sheet (you can't watch video);
iterate by timecode. The cut is just a concat list, so re-edits are seconds and cheap.

---

## ⚖️ Per-clip workflow (how it actually shipped)

1. **Agree the STORY first** — full-song storyboard by act → `out_memes/<slug>/storyboard.md`.
   Get story sign-off before generating anything.
2. **Generate EVERY shot as a separate RAW clip** — no audio, no lipsync, no assembly — into
   `out_memes/<slug>/video/shots/` named `NN_<id>_<desc>.mp4` (NN = order in the film, so
   Obsidian sorts them in story order). Seed images live in `video/`.
3. **User approves each clip individually** ("01–05 ok, 13 redo"). Re-roll only the rejects
   (re-gen seed → re-gen clip into the same filename). The singing close-ups stay un-lipsynced
   at this stage — approve the raw motion first.
4. **Only after ALL clips are approved** do the post: lyric-aligned concat + audio + lipsync.

Never silently fold a bad shot into the cut. Generate in background batches; present each as a
frame-strip. The "approve raw clips first, post once" order means a rejected shot costs one
re-roll, not a re-render of the whole film.

---

## 🎛 Choose the image + video models FIRST (ask the user — important)

Model choice is the user's call, per meme — do NOT silently hardcode it. At the start of the
video stage, run `hf model list --image` and `hf model list --video`, present a short menu, and
let the user pick **one image model + one video model**. Then use those throughout. Confirm price
with `hf generate cost <slug> …` and params with `hf model get <slug>` (duration enums & flags
differ per model). If the user has no preference, default to the ⭐ picks below.

**Image models** (Higgsfield):
- ⭐ `nano_banana_2` (Nano Banana Pro) — best identity-preserving reference edits (artist face →
  new scene), 16:9, up to 4k. ~2cr. The workhorse here.
- `text2image_soul_v2` (Higgsfield Soul V2) — signature cinematic photoreal.
- `flux_2` (FLUX.2) · `seedream_v4_5` (Seedream 4.5) — strong general photoreal / multi-ref.
- `gpt_image_2` (GPT Image 2) — best at **legible text/UI** → use it for phone-screen inserts.
- `recraft_v4_1`, `z_image`, `grok_image`, `nano_banana_flash` — alternates.

**Video models** (Higgsfield i2v):
- ⭐ `veo3_1` (Veo 3.1) — best cinematic motion/camera. `duration` **4/6/8** only. ~11/16.5/22cr.
- `kling3_0` (Kling 3.0) — lively faces, ~10cr/5s; `medias` = image inputs only.
- `seedance_2_0` (Seedance 2.0) — strong, but **HF moderation blocks it** for real-person likeness
  (`ip_detected`) and risqué outfits (`nsfw`) → avoid for an artist face / fishnets.
- `wan2_7` · `wan2_6` · `minimax_hailuo` · `grok_video_v15` — alternates.
- `cinematic_studio_video_3_5` · `soul_cast` — Higgsfield's director-preset style.

(The user may even mix per-shot — e.g. `gpt_image_2` only for the screen inserts, `nano_banana_2`
for everything else. Honor that.)

## Higgsfield CLI (image + video)

Binary: `higgsfield` (aliases `hf`, `higgs`). Uses the **logged-in account's credits**
directly — the **plus plan works**, no "ultra"/Cloud-API-key needed.

```bash
hf auth login                 # browser device login — the USER must run this interactively (`! hf auth login`)
hf account status             # email, plan, credits
hf model list [--image|--video]
hf model get <job_set_type>   # params + defaults
hf generate cost <type> [--param v]…    # estimate credits WITHOUT spending — do this before big batches
hf generate create <type> --prompt "…" --image ./seed.png --duration 6 --aspect_ratio 16:9 --wait
#   media flags: --image --start-image --end-image --video --audio (local paths auto-upload)
#   --wait blocks and prints the result URL on the last line:  grep -oE 'https://\S+\.(png|mp4)' | tail -1
```

Params reference for the ⭐ defaults (other models: `hf model get <slug>`):

**`nano_banana_2` (default image):** `prompt`(req), `aspect_ratio`
(auto/1:1/…/9:16/16:9/21:9), `input_images` (array, pass via `--image`), `resolution`
(1k/2k/4k, default 2k). **~2 credits.**

**`veo3_1` (default video):** `prompt`(req), `input_image` (via `--image`), `duration`
**4/6/8 only**, `aspect_ratio`, `quality` (basic/high/ultra). **~11 / 16.5 / 22 credits** for 4/6/8s.

---

## Lipsync — fal Sync 2.0 Pro (NOT in HF CLI)

The Higgsfield CLI has **no native lipsync** — its video models' `medias` accept images
only; `sound`/`generate_audio` just let the model invent its own audio. Their "Lipsync
Studio" is **web-only**. So lipsync runs on **fal** (same Sync.so engine HF uses):

```python
import fal_client
r = fal_client.subscribe("fal-ai/sync-lipsync/v2",
      arguments={"video_url": fal_client.upload_file(clip),
                 "audio_url": fal_client.upload_file(audio_slice),
                 "model": "lipsync-2-pro", "sync_mode": "cut_off"})
url = r["video"]["url"]
```
`lipsync-2-pro` = Sync.so Lipsync 2.0 (lip-replacement: edits only the mouth, keeps the
shot). **Best available** for "keep the cinematic shot + accurate singing." Needs
`$FAL_KEY`. Gotchas:
- Transient `FalClientHTTPError: Error scheduling request … try again later` → **retry with backoff**.
- Run the singing slots **in parallel** (`ThreadPoolExecutor(max_workers=3)`) — a full song has 5–9.
- **WATCH the fal balance** (`GET https://rest.alpha.fal.ai/billing/user_balance` with `Authorization: Key`):
  each lipsync ≈ $0.3–0.6; once it dips **below 0, fal returns 403 on everything** mid-run (storage
  token fails). Check before a big lipsync batch; top up.

---

## 🎯 Quality rules (learned the hard way)

- **Singing close-ups → LOCKED STATIC HEAD.** Generate with: *"locked-off static close-up,
  head still and facing camera, ONLY mouth/lips move, no head turn, no head rotation, no
  camera move."* A moving head makes lipsync **warp/rotate the face**. Then lipsync.
- **Walk-away shots → backs only.** *"filmed from behind, backs to camera the entire time,
  do NOT turn heads, faces never visible"* — otherwise Veo spins a face mid-walk.
- **Identity / likeness:** feed the artist's **real photo** as `--image` to `nano_banana_2`
  (grab a clean frontal portrait — source-video frames of a live show are usually
  head-up/eyes-closed; a web image search for a clear portrait works better).
- **White balance / consistency:** force clean neutral B&W across ALL clips at assembly with
  ffmpeg `hue=s=0` (Nano Banana sometimes adds a warm/sepia cast).
- **Establishing wides:** if the artist is far/small the face won't match — keep him
  medium or accept a silhouette; don't ask for a tiny recognizable face.
- **Seed over-anchoring:** Nano Banana Pro REPRODUCES a strong reference image almost verbatim.
  For a shot that's a big departure from the ref (a different character, a CU, an insert), a plain
  prompt yields a near-clone of the ref. Frame it as a transformation:
  *"Using the man from the reference (keep his face) — create a NEW scene: … NO woman, NO phone."*
- **Screens / UI / text in frame:** AI garbles letters & words. For a phone screen use **icons +
  digits only** ("hearts, bar-chart columns, like/comment counts as plain digits; absolutely NO
  text, NO letters, NO words"). A retro-Instagram feed (small photos + heart/comment icons + digit
  counts) reads great; on the i2v animation pin it ("screen content stays exactly the same, no scroll").
- **Rhythm:** vary shot lengths (e.g. 6/6/6/4/6/6/6/4/4), not a metronome. Trim in the edit.

---

## Assembly recipe (ffmpeg)

Lipsync the singing shots FIRST (each with its **audio sub-slice at the matching cut
offset**), then concat. Per clip: trim to its cut length, normalize to 1920×1080, fps 24,
neutral B&W.

```
vf = "hue=s=0,scale=1920:1080:force_original_aspect_ratio=increase,crop=1920:1080,fps=24,setsar=1"
# inputs: clip1..N + the audio segment ; filter:
#   [i:v]trim=0:<len>,setpts=PTS-STARTPTS,<vf>[vi];  …  [v0][v1]…concat=n=N:v=1:a=0[v]
# -map "[v]" -map "<N>:a" -t <total> -c:v libx264 -pix_fmt yuv420p -c:a aac
```
Audio slices: `seg.mp3 = audio[start:end]` for the whole act; per-singing-shot
`audio[cut_in:cut_out]` (e.g. shot at 0:28–0:34 → lipsync with `audio[28:34]`).

---

## Audio side (reference — see also work/memegen scripts & docs/audio-pipeline.md)

1. **Lyrics:** parody that **matches the original meter** (syllables + stress per line,
   pull word-level timings from the source `subs*.vtt`) AND is **de-plagiarized** (reword
   every distinctive source line — Suno's lyric filter blocks verbatim → `GEN_BLOCK`).
2. **Real minus:** `scripts/separate_minus.sh <orig> minus.wav` (BS-Roformer `ep_317`).
3. **Suno Add-Vocals over the minus:** upload `minus.mp3` → identify as *Instrument Stem* →
   Continue → paste lyrics + vocal style → Create (web/nodriver). **Suno uploads are
   rate-limited** — one upload, then wait; rapid retries escalate the block.
4. **Artist voice:** `scripts/rvc_voice.sh <suno_take_url> <rvc_model_zip> out.mp3`
   (Replicate `realistic-voice-cloning`; e.g. Tsoi = `varaslaw/victor_coy`). Needs
   `$REPLICATE_API_TOKEN`.

**Do NOT auto-open an audio player** — the user listens in Obsidian. Secrets live in
`~/.zshenv`, never in chat.

---

## See also (compose with these skills)

- **`higgsfield-generate`** — full Higgsfield model catalog & CLI details (images/video).
- **`falai`** — fal.ai REST for lipsync (`sync-lipsync/v2`) and any fal model.
- **`suno-cover-architect`** — copyright-clean Suno style prompts for new lyrics over a vibe.
- **`nodriver`** — drive Suno's web "Add Vocals" when the upload must be automated.

## Cost / credits sanity

`hf generate cost …` before any batch. Rough: image 2cr; Veo 6s ≈ 16.5cr; a 48s act
(~9 shots) ≈ ~140cr. Monthly plan credits don't roll over (use them). A full 3-minute song
≈ 30–36 shots ≈ ~450–550cr — confirm budget / do it by act.

---

## Lyric-aligned editing (whisper)

Don't eyeball the cut against the song — **transcribe and place shots on their words.**
- **mlx-whisper** (`mlx-community/whisper-large-v3-turbo`, Apple Silicon). Transcribe the
  **FULL MIX** (`audio.mp3`), NOT the isolated vocal stem — the bare stem hallucinates loops.
  Hallucination guard: `condition_on_previous_text=False`, `no_speech_threshold≈0.5`, and an
  `initial_prompt` listing the song's vocabulary. Also `silencedetect=n=-32dB:d=1.2` on the
  separated vocal gives the vocal-active spans for placing the singing close-ups.
- **Cut thematic shots ONTO their line:** «тушь и помада» → lipstick CU, «кольцевой свет» →
  ring-light, «лайки и донаты» → phone-hearts, «мамка зовёт» → she turns/runs. This is what
  makes it feel directed instead of random b-roll.
- **Never jump-cut two similar artist close-ups back-to-back** — a viewer reads it as a glitch.
  Insert a counter-shot (her reaction) between his singing takes (shot/counter-shot).
- The cut is a concat list → re-edits are **seconds and cheap** (clips & lipsyncs cached). Iterate
  with the user by timecode ("2:33–2:48 bad" → swap just that slot, re-concat).

## Publish → YouTube (composio)

The user's `composio` CLI drives their channel. **Web-dashboard connection ≠ local CLI session** —
run `! composio login` locally (browser device login), confirm `composio whoami` shows the real
email (not `pg-test-…`).
```bash
composio search "upload video to youtube"                          # → slugs (authed only)
composio execute YOUTUBE_GET_CHANNEL_STATISTICS -d '{"mine":true}' # confirm channel id == target
# big file → MULTIPART; --file injects the one uploadable arg, -d @json for the rest
composio execute YOUTUBE_MULTIPART_UPLOAD_VIDEO --file video/final.mp4 -d @payload.json
#   payload: {title, description, categoryId:"10"(Music), privacyStatus:"private", tags:[...]}
composio execute YOUTUBE_UPDATE_THUMBNAIL -d '{"videoId":"<ID>","thumbnailUrl":"<public url>"}'
```
- **API audit limit:** an unaudited API project (composio's) **forces uploads to `private`** and
  can't flip them public — the **user does the final Public flip in Studio**. The `videoId` hides
  in the response's thumbnail URL (`…/vi/<ID>/…`).
- **Thumbnail** wants a **public image URL** (not a local file) → resize to 1280×720 jpg and host on
  fal storage (`fal_client.upload_file`). Requires the **channel be verified** (`youtube.com/verify`)
  or it 403s ("doesn't have permissions to upload custom thumbnails").
- No API "pin comment" — hand the pinned comment to the user. AI-disclosure is a Studio "altered
  content" checkbox, not the description. Keep title/description/tags to the user's taste.

## Verification technique (you can't watch video)

Verify everything with **frame strips**: `ffmpeg -ss <t> -i clip -frames:v 1 f.png`, tile midframes
with `hstack`/`vstack`, then Read the PNG. Use it on seeds, every shot, lyric-aligned moments, and
cut **seams** (sample 33 / 33.5 / 34 / 34.5 s around a join to catch which side warps). zsh chokes on
`$((…))` inside inline loops and this ffmpeg build has **no `drawtext`** — build strips in a python heredoc.
