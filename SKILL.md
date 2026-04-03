---
name: github-growth-tracker
description: >
  Track GitHub repository growth metrics (stars, forks, issues, commits, subscribers) over time
  and generate periodic digests with trend analysis. Compare your repos against a watchlist of
  repos you care about. Requires GITHUB_TOKEN environment variable (GitHub fine-grained PAT with
  public repo read-only access). Use when: (1) User asks to check GitHub repo stats or growth,
  (2) User wants a GitHub growth digest or report, (3) User asks to track, add, or remove
  GitHub repos from monitoring, (4) User wants to compare their repos against specific other
  repos, (5) User mentions "github tracker", "repo growth", "star tracker", "repo analytics",
  "repo digest", "github watchlist", or similar. Also use when setting up scheduled GitHub
  repo monitoring via cron or heartbeat.
---

# GitHub Repo Growth Tracker

Track stars, forks, issues, and commit activity across GitHub repos. Generates text digests with trend arrows (↑↓→) and compares your repos against a watchlist you curate.

## Credentials

**Required:** GitHub Personal Access Token (fine-grained, public repo read-only).

Create at github.com → Settings → Developer settings → Personal access tokens → **Fine-grained tokens**:
- **Repository access:** Public repos (read-only)
- **Permissions:** Read access to repository metadata, commit status

This is narrower than the classic `repo` scope (which grants read/write to all repos including private).

**How it works:** Pass the token during `setup --token TOKEN` — it's saved to `~/.openclaw/credentials/github.json` (outside workspace, won't be backed up with repo data). All subsequent commands read from there automatically.

**Advanced:** Alternatively, set `GITHUB_TOKEN` as an environment variable (takes priority over the credentials file).

⚠ **Security note:** Token is stored plaintext on disk at `~/.openclaw/credentials/github.json`. This is standard practice for personal CLI tools (SSH keys, .env files, browser cookies are all plaintext at rest). For higher security, use the `GITHUB_TOKEN` environment variable instead.

## Setup Flow

When a user wants to start tracking their GitHub repos:

1. Ask for their **GitHub Personal Access Token** (see Credentials section above)
2. Run `setup` to list all their repos:
   ```
   GITHUB_TOKEN=<TOKEN> python3 scripts/github_tracker.py setup --token <TOKEN>
   ```
   The `--token` flag saves the token to `~/.openclaw/credentials/github.json` for future runs.
   If omitted, the script reads from the `GITHUB_TOKEN` env var.
3. Present the list and help them select which repos to track
4. Add repos (multiple at once):
   ```
   python3 scripts/github_tracker.py add owner/repo1 owner/repo2
   ```
5. Optionally add repos to compare against (watchlist):
   ```
   python3 scripts/github_tracker.py add other/repo --watch
   ```
6. Run `fetch` to record initial data:
   ```
   python3 scripts/github_tracker.py fetch
   ```

## Commands

All commands run from the skill directory.

| Command | Purpose |
|---------|---------|
| `setup [--token TOKEN]` | List user's repos. Optionally save token to credentials. |
| `fetch` | Fetch current metrics for all tracked + watched repos. |
| `digest` | Generate growth digest with trends and watchlist comparison. |
| `add repo1 [repo2 ...] [--watch]` | Add repos to tracking. `--watch` adds to watchlist. |
| `remove owner/repo` | Stop tracking (history preserved). |
| `list` | Show tracked repos and watchlist with latest stats. |

## Digest Output

Compact text report. If a watchlist exists, each tracked repo gets a one-line verdict comparing commit velocity against the watchlist average.

The agent formats the digest for the current channel:

**Terminal:** Use raw script output as-is.

**Slack/Discord:** Bold headings, blockquotes for metrics, monospace for language, `─────────` under title, `· · ·` separators between repos:
```
📊 *GitHub Growth Digest — Friday April 03, 2026*
─────────────────────────────────────

📌 *Your Repos*

*99rebels/blog-translator*
> ⭐ Stars: 1 NEW
> 🍴 Forks: 0 NEW
> 📋 Issues: 0 NEW
> 📝 Commits (4w): 5 ↑ (+5)
> `TypeScript` · Last push: 2026-04-03
> vs watchlist: ✅ above avg commit velocity

_· · · · · · · · · · · · · · · · · · · · · · · · · · ·_

*99rebels/leaving-cert-project*
> ⭐ Stars: 0 NEW
> 🍴 Forks: 0 NEW
> 📋 Issues: 0 NEW
> 📝 Commits (4w): 2 ↑ (+2)
> `Python` · Last push: 2026-03-25
> vs watchlist: ⚠ below avg commit velocity

_· · · · · · · · · · · · · · · · · · · · · · · · · · ·_

📌 *Watchlist*

*octocat/Hello-World*
> ⭐ 3,550 | 🍴 5,963 | 📋 3,833 | 📝 0

_· · · · · · · · · · · · · · · · · · · · · · · · · · ·_

*torvalds/linux*
> ⭐ 226,697 | 🍴 61,351 | 📋 3 | 📝 1,131
```

**WhatsApp:** Bold repo names only, line breaks between repos, compact watchlist. No blockquotes, no code formatting, no markdown tables. Use `─────` separators:
```
📊 GitHub Growth Digest — Friday April 03, 2026
─────────────────────────────────────

📌 Your Repos

*99rebels/blog-translator*
⭐ Stars: 1 NEW
🍴 Forks: 0 NEW
📋 Issues: 0 NEW
📝 Commits (4w): 5 ↑ (+5)
TypeScript · Last push: 2026-04-03
vs watchlist: ✅ above avg commit velocity

─────────────────────────────────────

*99rebels/leaving-cert-project*
⭐ Stars: 0 NEW
🍴 Forks: 0 NEW
📋 Issues: 0 NEW
📝 Commits (4w): 2 ↑ (+2)
Python · Last push: 2026-03-25
vs watchlist: ⚠ below avg commit velocity

─────────────────────────────────────

📌 Watchlist

*octocat/Hello-World*
⭐ 3,550 · 🍴 5,963 · 📋 3,833 · 📝 0

*torvalds/linux*
⭐ 226,697 · 🍴 61,351 · 📋 3 · 📝 1,131
```

**Discord:** Similar to Slack but no blockquotes (Discord ignores `>` on multi-line). Use `**bold**` headings, `` `code` `` for language, `─────` separators, blank lines between repos.

If the user asks for more detail about watchlist repos, read the history files in `data/github-tracker/repos/` and provide a fuller breakdown.

## Scheduled Monitoring

**Daily (fetch + digest):**
```
python3 <skill_path>/scripts/github_tracker.py fetch && python3 <skill_path>/scripts/github_tracker.py digest
```

No separate command needed for watchlist comparison — it's automatic in the digest.

## Config & Data

- **Credentials:** `~/.openclaw/credentials/github.json` (token, auto-created on setup)
- **Config:** `~/.openclaw/workspace/data/github-tracker/config.json` (repo lists only — no secrets)
- **History:** `~/.openclaw/workspace/data/github-tracker/repos/` (one JSON per repo, trimmed to 90 days)

Data lives outside the skill folder so it persists across skill updates. Removing a repo from tracking does not delete its history.

## Notes

- GitHub API rate limit: 5,000 req/hour authenticated
- `fetch` skips repos already recorded today (safe to run multiple times)
- Commit data uses the participation API (weekly buckets, last 52 weeks) — `commits_4w` is the sum of the last 4 weeks
- History is automatically trimmed to 90 days on each save
- `add` accepts multiple repos at once: `add repo1 repo2 repo3`
