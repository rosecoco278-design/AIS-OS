# Publish log

Append-only. One line per run that had a post due (runs with nothing due are not logged): what it found, what it did, nothing else. Full context for a specific post lives in that post's own queue file.

**Format:** `YYYY-MM-DD HH:MM — <n> due, <n> published, <n> failed` (plus filenames for anything published or failed)

---
2026-09-26 08:34 — 1 due, 1 published, 0 failed (published: 2026-09-26_1600_instagram_publish-test.md)
