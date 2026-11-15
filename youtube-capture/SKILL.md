---
name: youtube-capture
description: Turn a YouTube video into a structured Obsidian/markdown note — a timestamped topic-by-topic summary plus a deep-dive per topic with verbatim quotes — and save the full transcript beside it so every claim can be checked. Use when the user pastes a YouTube URL and asks to capture, summarize, digest or take notes on it.
---

# YouTube Capture

You fetch the transcript, write the note, save the transcript beside it, and run the fidelity audit. Do not ask permission before fetching.

## Settings (ask once, then reuse)

- **Notes folder** — where notes go. Use what the user says; otherwise `./youtube/` in the current project.
- **Deep-dive language** — default Thai (`th`). Use English or another language if the user asks.

`SKILL_DIR` below means the folder this file is in.

## 1. Read the input

- The YouTube URL(s) in the user's message. One video at a time; finish one before the next.
- Optional type tag: `#thinking` (ideas to reflect on) or `#building` (tools, how-tos to act on). If given, it becomes the `type:` field; otherwise leave `type:` out.
- Any note the user adds goes, verbatim, into `## My Notes`.

## 2. Fetch transcript + metadata

```bash
mkdir -p .yt-work
uv run "$SKILL_DIR/scripts/fetch_transcript.py" "<URL>" --transcript .yt-work/transcript.txt --metadata .yt-work/metadata.json
```

(`uv` installs the two Python packages it needs. Without `uv`: `pip install youtube-transcript-api yt-dlp`, then run it with `python3`.)

Read both files. Take `title`, `channel`, `duration_string`, `url` from the metadata — never leave `duration` blank.

**If YouTube answers 429 / "Too Many Requests" / blocked:** stop. Retrying makes the block longer. Tell the user to try again in a few hours.

## 3. Write the digest

**Long or course-style videos (> ~90 min, a lecture, an "N rules" rundown):** ask the user first whether they want a normal digest or a more study-grade breakdown. Don't decide alone.

**Walk the transcript start → end and list every distinct topic segment before writing.** Boundaries go where the topic changes, never every N minutes.

- **Summary:** one bullet per segment, each starting with its `[mm:ss]`. At least 5; a long conversation has roughly one per 3–5 minutes.
- **Deep dive:** one `##` section per summary bullet, separated by `---`. Heading in the video's language, naming what is actually interesting there (never "Introduction" / "Key ideas").

Each section:

```markdown
## <specific topic heading>
<one-sentence hook>

**Timestamp:**
* [mm:ss-mm:ss] <what happens in this stretch>

### Core idea
<the idea in 2–5 sentences, told like a friend explaining it — narrative, not a report>

### Breakdown & examples
<the speaker's OWN story/example that makes it land>

### Quotes & tips
- "<verbatim line from the transcript, inside this section's range>" [mm:ss]

**Actionable tips:**        ← only if the speaker actually gives advice here; otherwise delete this block
1. **<keyword>:** <the speaker's own advice> [mm:ss]
```

(When writing in Thai, use the headings `### คอนเซปหลัก`, `### Breakdown Idea & Examples`, `### Original Quotes & Actionable Tips`.)

### Source fidelity — non-negotiable

The transcript is saved beside the note, so every claim is checkable.

- **Nothing invented**: no tips, anecdotes, numbers, names or examples that are not in the transcript. When the speaker gives an example, use theirs.
- **Quotes are word-for-word.** 1–3 per section, chosen by importance. Copy exactly — no rewording, cleaning up or translating.
- **How the audit compares quotes:** it strips each line's `[mm:ss]`, joins lines with ONE space, collapses whitespace. So: a quote crossing a line break needs that space; quote inside one speaker turn (`>>` markers survive); never join two passages with `...` into one quote — write two quotes; don't paste `[mm:ss]` markers inside the quote text.
- **Every `[mm:ss]` you cite must be a real line in the transcript** — copy it, never round or guess.
- **A quote's `[mm:ss]` must fall inside its own section's range.** If it doesn't, move the quote or fix the section — never widen a range to swallow it.
- **Coverage:** the sections' ranges together must cover ≥ 90% of the runtime with no hole > 30 s. A hole means you skipped content: go read it and add a section.
- Every voice that isn't the narrator (clips, guests, a poem read aloud) gets covered — it's usually the evidence the video is built on. If two segments contradict each other on purpose, keep both.
- A section whose range runs 10+ minutes must cite something from each half.
- Write what was said, never sentences *about* the timestamp ("the key idea at 05:20 is…").

## 4. Pick 3 tags

Exactly 3 lowercase kebab-case concepts that could recur across other notes (e.g. `business-strategy`, `focus`). Never `thinking` / `building` — those go in `type:`.

## 5. Save

**Filename:** `<notes folder>/YYMMDD-<slug>.md` — `YYMMDD` = today; slug = short cleaned title (drop `| EP.xx`, `#tags`, `@channel` from the slug only).

```markdown
---
title: "<ORIGINAL video title, verbatim>"
url: "<url>"
channel: "<channel>"
platform: youtube
type: <thinking | building>        # omit if no type tag was given
captured: <YYYY-MM-DD>
status: captured
duration: "<mm:ss>"
tags: [<tag-1>, <tag-2>, <tag-3>]
---

# <ORIGINAL video title, verbatim>

> **Channel**: <channel> · **Duration**: <mm:ss>
> **URL**: <url>

---

## Summary

* [mm:ss] …

---

## Deep Dive

(sections from step 3)

## My Notes        ← only if the user wrote a note

---

## Source

- 🎬 [Watch on YouTube](<url>)
- 📄 Transcript (cross-check): [[_transcripts/<slug>|full transcript with timestamps]]
```

The note ends at `## Source`. **Never paste the transcript into the note.** Save it as a sibling instead, by copying the file (don't retype it):

```bash
mkdir -p "<notes folder>/_transcripts" && cp .yt-work/transcript.txt "<notes folder>/_transcripts/<YYMMDD-slug>.md" && rm -rf .yt-work
```

## 6. Audit (mandatory)

```bash
python3 "$SKILL_DIR/scripts/audit_fidelity.py" "<notes folder>/<YYMMDD-slug>.md"
```

It fails on: a quote not verbatim in the transcript, a `[mm:ss]` that doesn't exist, a quote outside its section's range, summary bullets ≠ sections, dropped runtime, and template-like filler. Fix and re-run until it prints clean. **Never present a note that fails.** An uncovered stretch means: go read it and add the missing section — never widen a range to make the number pass.

## 7. Report

Give the user the path to the note and a one-line summary of the video.
