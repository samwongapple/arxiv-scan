# arXiv daily scan — spin-qubit process-tensor project

Daily digest of new arXiv papers relevant to the project (process tensors,
quantum memory witnesses, DD noise spectroscopy, spin-qubit noise), with a
watchlist covering the nearest-neighbor groups from the research plan's risk
register (R1).

## What it does

- Queries the arXiv API for the latest submissions in `quant-ph` and
  `cond-mat.mes-hall` (last 2 days by default; papers appear once and won't
  duplicate across days if you run daily with `--days 2`, since the digest is
  per-day and overlap is harmless).
- Scores each paper against keyword groups and the author watchlist.
- Labels hits **CRITICAL** (watched author + on-topic, or very high keyword
  score), **RELEVANT**, or **FYI**, and writes `digests/YYYY-MM-DD.md`.

## Setup (GitHub Actions, ~5 minutes)

1. Create a repository (private is fine), e.g. `arxiv-scan`.
2. Add `arxiv_scan.py` at the repo root.
3. Add the workflow at `.github/workflows/arxiv-digest.yml`.
4. Push. Test it immediately via **Actions → arXiv daily digest → Run
   workflow** (the `workflow_dispatch` trigger).
5. Each weekday at 13:00 UTC a digest is committed to `digests/`. If a
   CRITICAL hit appears, the workflow opens a GitHub issue, which GitHub
   emails to you (configure notification settings if needed).

Notes:
- GitHub may pause scheduled workflows on repos with no activity for 60 days;
  the daily digest commits count as activity, so this self-sustains.
- Scheduled runs can lag a few minutes to hours at busy times; that's fine
  for this purpose.

## Local / cron alternative

```bash
python arxiv_scan.py --days 2            # writes digests/YYYY-MM-DD.md
python arxiv_scan.py --days 7 --stdout   # catch-up after a week away
```

Add to crontab for a daily 9am local run:
`0 9 * * 1-5 cd ~/arxiv-scan && python3 arxiv_scan.py --days 2`

## Tuning

All knobs are at the top of `arxiv_scan.py`:
- `WATCH_AUTHORS` / `WATCH_AUTHORS_STRICT` — the ambiguous surnames (White,
  Costa, Viola, Link) require an initial/first-name match; otherwise they are
  flagged only weakly (`name?`) and never exceed RELEVANT.
- `KEYWORD_GROUPS` — (points, phrases). A paper scores the sum over groups
  with at least one phrase present in title+abstract.
- `CRITICAL_SCORE` / `RELEVANT_SCORE` — thresholds (default 8 / 4).

Suggested cadence with Claude: once a week, paste the accumulated digests
(or just the CRITICAL/RELEVANT sections) into the project chat and ask for a
triage against the research plan — overlap assessment, differentiation
check, and whether any decision gate in the plan is triggered.

## Recommended pairing

Run this *and* a Claude scheduled task (Cowork, or Claude Code cloud tasks at
claude.ai/code/scheduled). The script is the reliable safety net with exact,
tunable matching; the Claude task adds judgment — summaries and project-aware
priority calls — on top of raw matching.
