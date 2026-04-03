# GitHub Growth Tracker

Track stars, forks, issues, and commit activity across GitHub repos. Generates periodic digests with trend analysis (↑↓→) and compares your repos against a watchlist you curate.

## Features

- **Track your repos** — stars, forks, open issues, subscribers, commit frequency
- **Watchlist comparison** — compare your commit velocity against repos you care about
- **Trend analysis** — see what changed with one-line arrows (↑↓→)
- **Channel-aware formatting** — clean output for Slack, Discord, WhatsApp, or terminal
- **Zero dependencies** — pure Python stdlib (no pip install needed)

## Install

```bash
clawhub install github-growth-tracker
```

## Quick Start

1. Create a [GitHub fine-grained PAT](https://github.com/settings/personal-access-tokens/new) with public repo read-only access
2. Run setup (your agent handles this):
   ```
   python3 scripts/github_tracker.py setup --token <TOKEN>
   ```
3. Add repos to track:
   ```
   python3 scripts/github_tracker.py add owner/repo1 owner/repo2
   ```
4. Add repos to your comparison watchlist:
   ```
   python3 scripts/github_tracker.py add other/repo --watch
   ```
5. Fetch and view your digest:
   ```
   python3 scripts/github_tracker.py fetch
   python3 scripts/github_tracker.py digest
   ```

## Commands

| Command | Purpose |
|---------|---------|
| `setup [--token TOKEN]` | List your repos, optionally save token to credentials |
| `fetch` | Fetch current metrics for all tracked + watched repos |
| `digest` | Generate growth digest with trends and watchlist comparison |
| `add repo1 [repo2 ...] [--watch]` | Add repos to tracking or watchlist |
| `remove owner/repo` | Stop tracking (history preserved) |
| `list` | Show tracked repos and watchlist |

## Scheduled Monitoring

Set up a daily cron or heartbeat to fetch and deliver digests automatically:

```
python3 <skill_path>/scripts/github_tracker.py fetch && python3 <skill_path>/scripts/github_tracker.py digest
```

## Credentials

Token is stored at `~/.openclaw/credentials/github.json` (outside workspace). Alternatively, set `GITHUB_TOKEN` as an environment variable.

## License

MIT-0
