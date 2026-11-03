# Trend-Gap Radar

[简体中文](README.md) | **English**

A daily CLI tool that compares topic engagement across global platforms (X, YouTube, Reddit) and Chinese platforms (Xiaohongshu, Douyin, Bilibili) to identify coverage gaps:

- Global → China: topics gaining traction overseas with low coverage on Chinese platforms
- China → Global: established Chinese topics and formats with low coverage overseas

The output is a Markdown brief containing heat scores for each side, a gap score, and links to the top 5 posts per side.

> Status: skeleton (v0.1). Topics are matched using English/Chinese keyword pairs in `topics.toml`. Unimplemented: LLM-based topic discovery and a China-lag-in-days metric (requires daily history).

## Scoring

Per-post engagement is calculated as a platform-weighted sum (e.g., X = `likes + 2×retweets + replies + views/100`) on a `log10` scale, where each point represents a 10× increase in engagement. Side heat is the mean score of the top 5 posts published within `window_days`.

`Gap = global heat − China heat`. A gap score of `≥ 1` indicates high activity overseas relative to China.

## Data and cost

Data is fetched through the [AIsa](https://aisa.one) REST API. A single API key covers all six platforms (Chinese platforms are served via TikHub). Measured cost is approximately $0.03 per topic per run, with a hard budget cap per run (default $0.5).

## Quick start

Requires Python 3.11+ with no third-party dependencies.

1. Create an AIsa API key: <!-- TODO: replace with Dub referral link and keep the disclosure -->https://aisa.one
   > Disclosure: This is a referral link. If you sign up and pay through it, the author earns a commission at no extra cost to you.
2. Export the key, run a free cost estimate, and execute a run:

```bash
export AISA_API_KEY=sk-aisa-...
python3 -m radar quote
python3 -m radar run
```

Reports are written to `reports/YYYY-MM-DD.md` and raw responses to `data/raw/YYYY-MM-DD/`. To rebuild a report from saved responses at no cost, run `python3 -m radar replay data/raw/<date>`.

## Boundaries

Queries public data only without platform logins. Reports output topic metrics and source links without redistributing post content. The API key is read only from the environment.

## License

MIT
