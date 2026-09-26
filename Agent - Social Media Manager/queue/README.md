# Publish queue

One file per scheduled post. The publish routine ([Social Media Manager - Publish Queue](https://claude.ai/code/routines/trig_011C2Gm74X83bKnc1tVLJnYp), `../../decisions/log.md` 2026-09-24, cron via the `schedule` skill, checks hourly at :58) only ever acts on files with `status: approved` whose `scheduled_time` has passed. Everything else it leaves untouched. It runs from a fresh GitHub clone, not local files — an approval only takes effect once pushed (the dashboard's Sync button, or a manual `git push`, does that).

## Lifecycle

`pending` → `approved` → `published` / `failed` / `rejected`

- **pending** — created when Social Media Manager recommends a schedule for content that's ready. Not yet cleared to post.
- **approved** — Nhi flips this (via the [agent dashboard](../../apps/agent-dashboard/README.md), by editing the file directly, or by telling Claude to after reviewing the content). This is the only status the cron routine acts on.
- **published** — set by the routine after a successful post. Never re-touched — the routine always checks status first, so a published item can't get posted twice.
- **failed** — set by the routine if publishing errored. Left for Nhi to review manually; the routine does not auto-retry a failed item (avoids silently repeated failures or duplicate posts from a retry racing a manual fix).
- **rejected** — set via the dashboard's Reject button (or manually). Means "reviewed, declined" — distinct from `failed`, which means the routine tried and hit a technical error. The routine treats both identically: it never acts on either.

Valid `account` values are the `alias` of a channel in `../channels.json` whose `mode` is `auto` and whose `platform` matches (today: `coolcat`, `ai-business-lab` for Instagram; `nhi-personal` for LinkedIn). The publish routine checks this: a post whose `account` does not match an automatic channel is marked `failed`, never sent to another account. `content:` names the content file the post belongs to, so A. Social Squad can show where each ticked channel stands. YouTube and Substack are by-hand channels and never appear here.

`substack` is never a valid `platform` value here — Substack is manual-only (see `../CLAUDE.md`). The routine ignores any file that names it, as a safety backstop, but one shouldn't get created in the first place.

## File format

One Markdown file per post, named `YYYY-MM-DD_HHMM_<platform>_<short-slug>.md` (e.g. `2026-09-25_0900_instagram_coolcat-launch-teaser.md`). Front matter + body:

```markdown
---
platform: instagram | linkedin
account: coolcat | nhi-personal | ai-business-lab   # an alias from ../channels.json
content: 2026-09-24-my-post.md
status: pending
scheduled_time: 2026-09-25T09:00:00+08:00
reminders: 1d, 1h
---

# Caption

The actual caption/commentary text, exactly as it should be posted.

# Media

https://a-public-url-meta-or-linkedin-can-fetch/image.jpg
```

Instagram requires `Media` to be a public `https://` URL with no query parameters (see `../CLAUDE.md` → Execution reality). LinkedIn can take a direct file reference instead — note that inline if it applies.

After the routine acts, it appends these fields to the front matter rather than editing anything else: `published_at`, and either the platform's post ID/permalink (success) or `error` (failure).

A run gets one line in `log.md` only if a post was due (published, failed, or skipped for a stated reason). A run with nothing due leaves no trace in the repo: no log line, no commit, no notification. The routine's run list on claude.ai already records every run. (Changed 2026-09-24: logging empty runs meant a commit and push every hour, which is what kept failing and notifying when GitHub access was missing.)

## Reminders (optional)

`reminders: 1d, 2h` is how long before `scheduled_time` you want to be nudged. Comma-separated, units `m`, `h` or `d`, up to 8 per post, each between 1 minute and 30 days. There is no default: leave the line out and there are no reminders. Clearing a post's reminders in the dashboard removes the line.

- Adjust it in the [agent dashboard](../../apps/agent-dashboard/README.md) Queue tab (chips on each pending or approved post, plus a custom time), or edit the line by hand. The dashboard rewrites only this line.
- The dashboard lists what is due or overdue and sends a browser notification, but only while its page is open. It cannot reach you when it is closed. A post that is past its time while still pending or approved is flagged Overdue whether or not you set reminders, because a pending post past its time will never publish.
- Social Media Manager uses the same lead times for the Google Calendar event on that post (one popup per lead time), so the calendar and the dashboard agree. With no `reminders:` line it adds no reminder of its own. See `../CLAUDE.md`.
- The publish routine never reads this field, so changing it cannot affect what gets posted or when.
- Only pending and approved posts have reminders. Published, failed and rejected posts don't.

See `_template.md` for a blank starting point.
