# ALP Analytics — Methodology

## File paths and fields

### Task file
```
src/apps/www-site/user-data/owners/<owner>/projects/<projectId>/tasks/<assemblyLineId>.json
```

Key fields:
- `linkedStreamIds` — array of session IDs, in run order
- `agents` — agent names ordered by station
- `stageResults` — object keyed by stage name (`todo`, `research`, `planning`, `implementation`, `testing`)
- `lastJobConclusion` — `"success"` | `"timeout"` | `"error"`
- `lastJobSessionId` — session ID of the final attempt

### Session directory
```
src/apps/www-site/user-data/sessions/<sessionId>/
```

#### activities.jsonl
One JSON object per line. Each line has:
- `timestamp` (Unix ms) — when the event occurred
- `type` — e.g. `"tool_use"`, `"message"`, `"stop"`

**Wall time formula:**
```
wallTime = lastLine.timestamp - firstLine.timestamp
```
Convert to minutes: `wallTime / 60000`.

#### tool-uses.jsonl
One JSON object per line. Each line has:
- `toolName` — e.g. `"Bash"`, `"Read"`, `"Write"`, `"Agent"`, `"AskUserQuestion"`, `"Skill"`
- `toolUseId` — unique ID
- `input` — tool input object
- `startedAt` (Unix ms)

**Tool distribution:**
```bash
# Count by tool name (jq one-liner)
cat tool-uses.jsonl | jq -r '.toolName' | sort | uniq -c | sort -rn
```

#### tool-use-ends.jsonl
One JSON object per line. Each line has:
- `toolUseId` — matches `tool-uses.jsonl`
- `durationMs` — how long the tool call took
- `exitCode` (for Bash calls)

**Slowest tool calls:**
```bash
cat tool-use-ends.jsonl | jq -r '[.durationMs, .toolUseId] | @tsv' | sort -rn | head -10
```

#### transcript.jsonl
Full conversation log. Each line has:
- `role` — `"user"` | `"assistant"`
- `content` — message content (may be array of content blocks)

**Find error messages:**
```bash
grep -i "error\|failed\|timeout\|ETIMEDOUT\|EADDRINUSE\|deadlock" transcript.jsonl | head -20
```

**Find pivot moments (when the model changed approach):**
```bash
grep -i "let me try\|instead\|pivot\|alternative\|workaround" transcript.jsonl | head -20
```

---

## Bottleneck categories

### Category A — Q&A overhead
**Signal:** High `AskUserQuestion` count (≥3 per station).  
**Impact:** Each call is an answer-gated round-trip. Even with auto-injection, sequential Q&A adds 2–4 min per question in pipeline context.  
**Fix direction:** Batch input mode, pre-filled parameter files, or `--confirm-all` flags in skills.

### Category B — Port/resource conflicts
**Signal:** `EADDRINUSE` errors in transcript, repeated port-allocation Bash calls.  
**Impact:** 5–15 min per retry cycle until a free port is found or the agent gives up.  
**Fix direction:** Use dynamic port allocation (`--environment test` in Aspire, or `port: 0` patterns).

### Category C — Interactive CLI in headless context
**Signal:** A Bash call hangs (no `tool-use-ends` entry within 5+ min of `tool-uses` entry), often followed by a timeout conclusion.  
**Pattern:** `aspire agent init`, `npx create-*`, or other CLIs that prompt for input even when given `--non-interactive` flag.  
**Fix direction:** Replace CLI call with direct file writes (e.g. write `.mcp.json` manually instead of running `aspire agent init`).

### Category D — Sequential work that could be parallel
**Signal:** Multiple `Agent` calls in series where outputs don't depend on each other.  
**Impact:** N × avg_agent_time vs avg_agent_time (parallel batch).  
**Fix direction:** Batch independent Agent calls in a single message using parallel tool use.

### Category E — Retry loops on transient failures
**Signal:** Same tool name repeated 3+ times in sequence with same or similar input; each attempt fails.  
**Impact:** Wastes 5–20 min depending on timeout per attempt.  
**Fix direction:** Add explicit retry limits in prompts; use `aspire wait <resource>` before dependent calls.

### Category F — Large output / token pressure
**Signal:** Very high `Write` counts with large files; model starts truncating or skipping sections late in session.  
**Impact:** Incomplete deliverables, requiring a re-run.  
**Fix direction:** Split large generation tasks across multiple Agent calls with scoped outputs.

---

## Classifying a stuck session

A session is "stuck" (not just slow) when:
1. `activities.jsonl` shows no new events for >5 min mid-session, AND
2. The final conclusion is `timeout` rather than `success`, AND
3. `transcript.jsonl` ends with the model waiting for tool output (last role is `"assistant"` with an open tool_use block)

Common stuck patterns:
- **PTY deadlock**: model invoked an interactive CLI; tool-use-ends never arrives
- **TUI injection gap**: `AskUserQuestion` answer was injected but no Enter keypress was sent; TUI stays on highlighted option
- **Network timeout**: external API call hung; no retry logic

---

## Timing a specific bottleneck

To measure how long a single tool call took:

1. Find its `toolUseId` in `tool-uses.jsonl`
2. Look up the same ID in `tool-use-ends.jsonl` → read `durationMs`
3. If no matching entry exists in `tool-use-ends.jsonl`, the call never completed (PTY deadlock or process kill)

To measure Q&A wait time:
- Find `AskUserQuestion` entries in `tool-uses.jsonl`
- Match to `tool-use-ends.jsonl` by ID
- `durationMs` includes the full round-trip (question sent → answer injected → Enter confirmed)

---

## Projecting runtime after fixes

For each bottleneck fix, estimate:
- **Current cost** — actual measured time from session files
- **Post-fix cost** — expected time after fix (e.g. 0 for eliminated Q&A, ~2 min for parallel agent batch)
- **Savings** — difference

Sum all savings and express as % of success-path total wall time.
