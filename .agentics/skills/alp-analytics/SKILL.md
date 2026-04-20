---
name: alp-analytics
description: Analyze a completed assembly line run — locate session files, compute per-station timings and tool distributions, identify bottlenecks, and write a report to analytics/.
---

Analyze a completed ALP (Assembly Line Pipeline) run and write a structured report to `analytics/`.

## Quick start

```
/alp-analytics <assemblyLineId>
```

`assemblyLineId` is the UUID shown in the assembly line URL, e.g. `d5550ff4-62b2-415e-9a08-e411bb6bef7c`.

## Steps

### 1. Locate the task file

Read the task JSON to get linked session IDs and station results:

```
src/apps/www-site/user-data/owners/<owner>/projects/<projectId>/tasks/<assemblyLineId>.json
```

The task JSON contains:
- `linkedStreamIds` — ordered list of session IDs (one per station attempt)
- `stageResults` — per-stage outcome summaries already written by the runner
- `lastJobConclusion` — final conclusion (`success` | `timeout` | `error`)

If you do not know `<owner>` or `<projectId>`, search for the file:

```bash
find src/apps/www-site/user-data/owners -name "<assemblyLineId>.json" 2>/dev/null
```

### 2. Locate session directories

Each session ID maps to a directory:

```
src/apps/www-site/user-data/sessions/<sessionId>/
```

Key files inside each session directory (read in this order):

| File | Purpose |
|---|---|
| `activities.jsonl` | Timestamped activity events — use for wall-clock timing |
| `tool-uses.jsonl` | Every tool call with `toolName`, `input`, `startedAt` |
| `tool-use-ends.jsonl` | Completion events with `durationMs` per tool call |
| `transcript.jsonl` | Full conversation — use to read error messages and pivots |

See `reference/analysis-methodology.md` for field names and timing formulas.

### 3. Analyze each station

For each session (= one station attempt):

1. **Calculate wall time** — `lastActivity.timestamp - firstActivity.timestamp` in ms → convert to minutes
2. **Count tool calls by name** — group `tool-uses.jsonl` by `toolName`, sort descending
3. **Identify error patterns** — grep `transcript.jsonl` for `error`, `failed`, `ETIMEDOUT`, `EADDRINUSE`, `deadlock`, `timeout`
4. **Flag AskUserQuestion overhead** — count `AskUserQuestion` calls; each is an answer-gated round-trip that adds latency
5. **Detect retries** — repeated identical tool names in sequence indicate a stuck loop
6. **Classify conclusion** — `success`, `timeout` (station hit wall-clock limit), or `error` (explicit failure)

See `reference/analysis-methodology.md` for bottleneck categories and how to spot them.

### 4. Build the pipeline summary table

After analyzing all stations, produce a summary table:

| # | Station | Session | Duration | Tool Calls | Result |
|---|---|---|---|---|---|
| 01 | … | … | …min | … | success/TIMEOUT/error |

If a station was retried, label attempts `04a`, `04b`, etc. Calculate:
- **Total wall time** (all attempts, including failures)
- **Success-path wall time** (only the attempts that fed the next station)
- **Wasted time** = difference, expressed as % overhead

### 5. Write the report

Write to `analytics/YYYY-MM-DD-<slug>.md` using the structure in `reference/report-template.md`.

Include for each station:
- What happened (narrative summary)
- Tool distribution table
- Bottleneck classification (use categories from methodology)
- Improvement suggestions with priority (P0/P1/P2) and estimated time savings

End with a **Bottleneck Ranking** table and **Projected runtime after fixes**.

See `analytics/2026-04-19-scarfold-run.md` as a concrete example output.

### 6. Commit

```bash
git add analytics/
git commit -m "analytics: <project> run <date>"
```

## Reference files

- `reference/analysis-methodology.md` — file field names, timing formulas, bottleneck categories, how to classify stuck patterns
- `reference/report-template.md` — fill-in markdown template matching the established report structure
