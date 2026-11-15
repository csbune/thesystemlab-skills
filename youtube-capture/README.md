# YouTube Capture — Claude Code skill

**ไทย** · เปลี่ยนวิดีโอ YouTube เป็นโน้ต Markdown/Obsidian ที่สรุปเป็นหัวข้อพร้อม timestamp และเจาะลึกทีละหัวข้อ ใช้คำพูดจริงของผู้พูด (ไม่แต่งเติม) และเก็บ transcript ไว้ข้างโน้ตเพื่อตรวจย้อนได้ทุกประโยค มีสคริปต์ตรวจความถูกต้องให้อัตโนมัติ

**English** · Turns a YouTube video into a markdown/Obsidian note: a timestamped topic summary, a deep-dive per topic built from the speaker's own words (nothing invented), and the full transcript saved beside it so every line can be checked. An audit script enforces it.

## ติดตั้ง · Install

1. Install [Claude Code](https://claude.com/claude-code) and [uv](https://docs.astral.sh/uv/) (`curl -LsSf https://astral.sh/uv/install.sh | sh`).
2. Copy this folder to `~/.claude/skills/youtube-capture/` (all projects) or `.claude/skills/youtube-capture/` (one project).
3. Restart Claude Code.

## ใช้งาน · Use

```
/youtube-capture https://www.youtube.com/watch?v=XXXXXXXXXXX
/youtube-capture <url> #building บันทึกไว้ทำต่อ
```

- โน้ตบันทึกที่ `./youtube/` (บอก Claude ได้ถ้าอยากเปลี่ยนโฟลเดอร์) · Notes go to `./youtube/` unless you name another folder.
- ส่วนเจาะลึกเป็นภาษาไทยโดยค่าเริ่มต้น · The deep-dive is written in Thai by default; ask for English and you get English.
- `#thinking` = เรื่องให้คิด · `#building` = เรื่องให้ลงมือทำ (optional).

## ได้อะไร · What you get

```
youtube/
  261115-short-title.md          ← the note
  _transcripts/261115-short-title.md   ← the transcript, for cross-checking
```

## หมายเหตุ · Notes

- If YouTube blocks requests (HTTP 429), wait a few hours — retrying makes it worse.
- Videos without captions can't be captured.

Made by **The System Lab** · https://thesystemlab.pages.dev · MIT licence
