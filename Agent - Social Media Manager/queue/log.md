# Publish log

Append-only. One line per run that had a post due (runs with nothing due are not logged): what it found, what it did, nothing else. Full context for a specific post lives in that post's own queue file.

**Format:** `YYYY-MM-DD HH:MM — <n> due, <n> published, <n> failed` (plus filenames for anything published or failed)

---
2026-09-26 08:34 — 1 due, 1 published, 0 failed (published: 2026-09-26_1600_instagram_publish-test.md)
2026-09-30 03:03 — 1 due, 1 published, 0 failed (published: 2026-09-30_1558_facebook_hello-facebook.md)
2026-09-30 09:59 — 2 due, 2 published, 0 failed (published: 2026-09-30_1758_facebook_where-to-start.md, 2026-09-30_1758_instagram_where-to-start.md)
