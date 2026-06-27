# astroph2discord

Deliver new **arXiv astro-ph** papers that match your keywords to a **Discord**
channel, automatically, every day — for free via GitHub Actions.

This is a Discord-focused successor to
[arXiv-owl](https://github.com/Y-Masayuki/arXiv-owl). It differs in two ways:

- **Official arXiv API** (`export.arxiv.org/api`) instead of scraping the
  advanced-search HTML, so it does not break when arXiv changes its page layout.
- **Discord webhook** delivery (rich embeds, message chunking) instead of Slack.

## How it works

1. Query the arXiv API for papers submitted to your chosen `astro-ph`
   subcategories in the last *N* days (newest first).
2. Score each paper: sum the weights of the keywords found in its title +
   abstract (case-insensitive substring match).
3. Post papers scoring at/above the threshold to Discord as embeds, one
   message per 10 papers (Discord's per-message embed limit).

## Setup

### 1. Create a Discord webhook

In Discord: **Server Settings → Integrations → Webhooks → New Webhook**, pick
the target channel, then **Copy Webhook URL**. The URL looks like
`https://discord.com/api/webhooks/<id>/<token>`.

### 2. Configure keywords

Edit [`config.yaml`](config.yaml). Add/remove keywords and adjust weights, the
`categories` you watch, the look-back window `days`, and `score_threshold`.

### 3. Run it automatically (GitHub Actions — recommended)

Fork/push this repo to your account, then add the webhook URL as a secret:

**Repo → Settings → Secrets and variables → Actions → New repository secret**
- Name: `DISCORD_WEBHOOK_URL`
- Value: your webhook URL

The workflow in [`.github/workflows/arxiv2discord.yml`](.github/workflows/arxiv2discord.yml)
runs daily (23:00 UTC = 08:00 JST). You can also trigger it manually from the
**Actions** tab ("Run workflow"), optionally overriding the look-back days.

> Note: arXiv does not announce on weekends. A daily run with `days: 1` is
> simplest and never duplicates papers (each run covers a disjoint 24 h window).
> If you prefer weekday-only runs, change the cron and bump `days` on Mondays.

### 4. Run it locally (optional)

```bash
pip install -r requirements.txt
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/…"

python arxiv2discord.py                 # use config.yaml defaults
python arxiv2discord.py --days 3        # look back 3 days
python arxiv2discord.py --days 3 --dry-run   # print payload, send nothing
```

`--dry-run` prints the exact JSON that would be sent to Discord — handy for
tuning keywords without spamming your channel.

## Configuration reference

| Key               | Meaning                                                        |
| ----------------- | -------------------------------------------------------------- |
| `categories`      | arXiv categories to query (default: all six `astro-ph.*`).     |
| `days`            | Look-back window in days. `1` = last 24 h.                     |
| `max_results`     | Safety cap on papers pulled from the API per run.              |
| `score_threshold` | Minimum summed score required to notify.                       |
| `keywords`        | `keyword: weight` map; scored against title + abstract.        |

## Notes & limitations

- The arXiv API requires a non-default `User-Agent`; this tool sets one. If you
  reuse the code elsewhere, keep a descriptive UA or arXiv returns `403`.
- Matching is plain substring matching (so `disk` also matches `disks`). For
  whole-word matching you'd need to switch `score_article` to a regex.
- There is no persistent "already seen" store; duplicate avoidance relies on the
  disjoint daily window. If you run more frequently, add an ID cache.

## Credits

- [Y-Masayuki/arXiv-owl](https://github.com/Y-Masayuki/arXiv-owl)
- [jinshisai/arXiv-owl](https://github.com/jinshisai/arXiv-owl)
- [fkubota/Carrier-Owl](https://github.com/fkubota/Carrier-Owl) (original idea)

## License

MIT — see [LICENSE](LICENSE).
