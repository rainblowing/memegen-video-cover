# memegen-video-cover

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) **skill** that produces a
high-quality AI **video-meme cover** end to end — a parody music video where a chosen artist
performs reworked lyrics over a soundalike of a famous song, with a cinematic regenerated video,
lipsync, and one-command publish to YouTube.

It's a **dynamic, human-in-the-loop workflow** (no orchestration framework): you steer it shot by
shot; it generates in background batches and shows you frame-strips to approve.

## Pipelines (you pick one at the start)

Step 0 of the workflow: choose the pipeline for the cover you're making — it decides the
lyrics + audio recipe; the video / assembly / publish stages are shared.

| | Pipeline | What it makes |
|--|---|---|
| **A** ⭐ | **Parody on the real minus** | NEW lyrics over the original song's actual instrumental (BS-Roformer minus → Suno "Add Vocals" → RVC) |
| **B** | **Voice swap** | The original lyrics & melody in another artist's voice (RVC on the separated vocal stem — no Suno) |
| **C** | **Soundalike from scratch** | A new song "in the vibe of" — Suno full generation → RVC, no original audio used |

The menu is open-ended — new cover types get a new letter.

```
STEP 0   → pick pipeline A / B / C (decides lyrics + audio)
LYRICS   → A: meter-locked parody, de-plagiarized · B: keep originals · C: free-form
AUDIO    → per pipeline (see SKILL.md) — all end in the RVC artist voice
VIDEO    → storyboard the whole song → a SEED image per shot → a CLIP per shot → you approve each
ASSEMBLE → transcribe the vocal (whisper) → cut shots ONTO their lyric lines → lipsync the singers → concat
PUBLISH  → composio → YouTube (upload + thumbnail)
```

**Stack:** Suno (vocals) · Replicate RVC (artist voice clone) · `audio-separator` BS-Roformer
(real minus) · Higgsfield (Nano Banana Pro images + Veo 3.1 / Kling / Wan / Seedance video —
**you pick the models**) · fal Sync 2.0 Pro (lipsync) · mlx-whisper (lyric alignment) · ffmpeg
(edit) · Composio (YouTube).

## First run (shipped)

Kino «Восьмиклассница» → «Онлифанщица» — Viktor Tsoi's voice on the real Kino minus, a 3:03
Soviet-noir / Blade-Runner video, 38 lyric-aligned cuts, 9 lipsync slots, satire on influencer
culture. → https://youtu.be/6ZnOKy3FevA

## Install

```bash
git clone https://github.com/rainblowing/memegen-video-cover ~/.claude/skills/memegen-video-cover
```
Then in Claude Code just say **"memegen-video-cover"** (or describe a meme-cover task) and the
skill loads. It composes with the `higgsfield-generate`, `falai`, `suno-cover-architect`, and
`nodriver` skills.

The skill assumes API keys in your env (`$FAL_KEY`, `$REPLICATE_API_TOKEN`) and authenticated CLIs
(`higgsfield auth login`, `composio login`). It never writes secrets to the repo.

## Note

This is a **parody / satire** tool. Respect copyright (Content ID can flag a soundalike melody)
and likeness rights; disclose AI-generated/altered content per the platform's rules.

## License

MIT — see [LICENSE](LICENSE).
