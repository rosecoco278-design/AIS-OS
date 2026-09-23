# Publish queue

One file per scheduled post. The publish routine ([Social Media Manager - Publish Queue](https://claude.ai/code/routines/trig_011C2Gm74X83bKnc1tVLJnYp), `../../decisions/log.md` 2026-09-24, cron via the `schedule` skill, checks hourly at :58) only ever acts on files with `status: approved` whose `scheduled_time` has passed. Everything else it leaves untouched. It runs from a fresh GitHub clone, not local files — an approval only takes effect once pushed (the dashboard's Sync button, or a manual `git push`, does that).

## Lifecycle

`pending` → `approved` → `published` / `failed` / `rejected`

- **pending** — created when Social Media Manager recommends a schedule for content that's ready. Not yet cleared to post.
- **approved** — Nhi flips this (via the [agent dashboard](../../apps/agent-dashboard/README.md), by editing the file directly, or by telling Claude to after reviewing the content). This is the only status the cron routine acts on.
- **published** — set by the routine after a successful post. Never re-touched — the routine always checks status first, so a published item can't get posted twice.
- **failed** — set by the routine if publishing errored. Left for Nhi to review manually; the routine does not auto-retry a failed item (avoids silently repeated failures or duplicate posts from a retry racing a manual fix).
- **rejected** — set via the dashboard's Reject button (or manually). Means "reviewed, declined" — distinct from `failed`, which means the routine tried and hit a technical error. The routine treats both identically: it never acts on either.

`substack` is never a valid `platform` value here — Substack is manual-only (see `../CLAUDE.md`). The routine ignores any file that names it, as a safety backstop, but one shouldn't get created in the first place.

## File format

One Markdown file per post, named `YYYY-MM-DD_HHMM_<platform>_<short-slug>.md` (e.g. `2026-09-25_0900_instagram_coolcat-launch-teaser.md`). Front matter + body:

```markdown
---
platform: instagram | linkedin
account: coolcat | nhi-personal
status: pending
scheduled_time: 2026-09-25T09:00:00+08:00
---

# Caption

The actual caption/commentary text, exactly as it should be posted.

# Media

https://a-public-url-meta-or-linkedin-can-fetch/image.jpg
```

Instagram requires `Media` to be a public `https://` URL with no query parameters (see `../CLAUDE.md` → Execution reality). LinkedIn can take a direct file reference instead — note that inline if it applies.

After the routine acts, it appends these fields to the front matter rather than editing anything else: `published_at`, and either the platform's post ID/permalink (success) or `error` (failure).

Every run — successful, failed, or a no-op because nothing was due — gets one line in `log.md`.

See `_template.md` for a blank starting point.
