# Spec — Programmatic Gate & Transition Tool Runner

**Status:** Draft — schema defined, execution **not** enabled.
**Owner:** Platform (Agentic Live).
**Last updated:** 2026-04-24.

Gates today are binary human-approval switches (`requiresApproval: true`). This spec extends gates — and, symmetrically, transitions — with the ability to run a **tool** (a deterministic, non-agentic process) and decide pass/fail/warn from its output. The immediate motivating use case is running `@google/design.md lint DESIGN.md` between the Design System station and Brand Deliverables station, but the shape is generic.

> **This document defines the contract only.** Execution of the `tool` block is intentionally deferred — the executor's location (station operator vs. platform sidecar vs. cloud runner) needs a separate decision and is out of scope for this draft.

---

## Why

1. **Contract enforcement.** A station hands off an artifact (DESIGN.md, a test report, a build output). Human approval is slow and subjective for things that can be checked mechanically.
2. **Determinism at the seam.** Agents are probabilistic; gate checks should be deterministic. A CLI exit code + structured output is the cleanest way to represent "this work meets the bar."
3. **Visibility to the project owner.** Findings (warnings, errors, summary) become a first-class artifact on the gate card in the UI, not buried inside a broadcast transcript.
4. **Re-usable primitive.** The same schema covers: design-md lint, `tsc --noEmit`, `dotnet test`, `playwright test`, `axe`, ffprobe, any JSON-emitting tool.

---

## Schema — `tool` block inside a gate

Extends the existing gate object. All fields besides `kind`, `command`, `args`, and `outcome` are optional.

```jsonc
{
  "id": "design-md-lint-gate",
  "name": "Design System Lint Gate",
  "description": "...",
  "requiresApproval": false,

  // NEW — programmatic tool definition
  "tool": {
    "kind": "cli",                  // enum: "cli" | "http" | "mcp" | "agent" (future)
    "runner": "node",               // informational; executor chooses actual runtime
    "command": "npx",
    "args": ["-y", "@google/design.md@0.x", "lint", "DESIGN.md", "--format", "json"],
    "cwd": "{{project.workspaceDir}}",
    "env": {                        // optional; only whitelisted keys flow through
      "LOG_LEVEL": "info"
    },
    "stdin": null,                  // or a string / { file: "path" }
    "timeoutSeconds": 60,
    "retries": { "max": 0, "backoffSeconds": 0 },

    "outcome": {
      "parse": "json",              // enum: "json" | "text" | "exitCodeOnly"
      "successOn": { "exitCode": 0 },
      "passIf":    "summary.errors == 0",
      "warnIf":    "summary.warnings > 0",
      "failIf":    "summary.errors > 0",
      "surface": {                  // which fields of the parsed output are shown in the UI
        "findings": "findings[]",
        "summary":  "summary"
      }
    },

    "caching": {                    // optional; executor MAY cache
      "keyInputs": ["DESIGN.md"],   // files whose hash is part of the cache key
      "ttlSeconds": 3600
    }
  },

  "enabled": false,                 // master switch — when false, tool block is metadata only
  "notes": "Human-readable note about why this gate exists / known gotchas."
}
```

### Field reference

| Field | Required | Purpose |
|---|---|---|
| `tool.kind` | yes | Kind of thing to run. Start with `cli`; leave room for `http`, `mcp`, `agent`. |
| `tool.runner` | no | Hint to the executor (`node`, `bun`, `dotnet`, `bash`). Not authoritative. |
| `tool.command` + `tool.args` | yes (for `cli`) | argv-style, not shell-escaped strings. Executor never invokes a shell. |
| `tool.cwd` | no | Working directory. Templated against the transition context. |
| `tool.env` | no | Environment variables. Only keys listed in `platform.envAllowlist` are forwarded; others are rejected at registration time. |
| `tool.stdin` | no | Literal string or `{ file: "relative/path" }`. |
| `tool.timeoutSeconds` | yes | Hard timeout. Exceeding → `failed` with `reason: "timeout"`. |
| `tool.retries` | no | Only applied on *infrastructure* failure (non-zero exit isn't retried). |
| `tool.outcome.parse` | yes | How to interpret stdout. `exitCodeOnly` means no parsing. |
| `tool.outcome.successOn` | yes | Short-circuit: if not met, result is `failed` regardless of `passIf`/`warnIf`. |
| `tool.outcome.passIf` / `warnIf` / `failIf` | ≥1 | Simple expression language (see below) evaluated against the parsed stdout. |
| `tool.outcome.surface` | no | JMESPath-like projections shown on the gate card in the UI. |
| `tool.caching` | no | Executor-level optimization. |
| `enabled` | yes | If `false`, the runtime treats the gate as a no-op pass and logs the reason. |

### Expression language (`passIf` / `warnIf` / `failIf`)

Intentionally minimal — do **not** adopt a full JS/JSONata runtime. Supported:

```
<path> <op> <literal>
<path> <op> <path>
<expr> && <expr>
<expr> || <expr>
!(<expr>)
```

Where:
- `<path>` is dotted JSON path with `[]` for array size (`findings[]` means `findings.length`).
- `<op>` ∈ `== != < <= > >= in contains`.
- `<literal>` ∈ number | quoted string | `true` | `false` | `null`.

If the expression can't be evaluated (missing field, type mismatch), treat it as `false` and flag `evaluationError` in the result. Never throw.

### Templating

`tool.cwd`, `tool.args[*]`, `tool.env.*` values support `{{ ... }}` templating against a **frozen** context, computed at transition time:

| Variable | Meaning |
|---|---|
| `project.id`, `project.owner`, `project.name`, `project.workspaceDir` | Project identity and local path |
| `task.id`, `task.title` | The task crossing the transition |
| `stage.from`, `stage.to` | Stage identifiers |
| `gate.id` | This gate |
| `run.id`, `run.startedAt` | Execution metadata |

Unknown variables are rejected at registration (linting the schema), not silently left in place.

---

## Lifecycle — what happens at a transition

1. **Trigger.** Station reports success and requests advance to the next stage.
2. **Gate resolution.** Platform looks up every gate on this transition in `.agentics/transitions.json`.
3. **Pre-flight.** For each gate:
   - If `requiresApproval: true` → open an approval card for the project owner (unchanged behavior).
   - If `tool` is defined and `enabled: true` → dispatch to the tool runner.
   - If both are present → tool runs first; approval card only opens if tool result is `warn` or if gate config sets `approveWarnings: true`.
4. **Execution.** Runner:
   - Renders templates against the frozen context.
   - Starts the process with resolved `cwd`, filtered `env`, timeout.
   - Captures `stdout` (up to a cap, e.g., 4 MiB), `stderr`, `exitCode`, `durationMs`.
5. **Parse.** If `parse: "json"`, attempts `JSON.parse(stdout)`. Parse error → `failed` with `reason: "parse-error"`, stderr surfaced.
6. **Evaluate.** Apply `successOn` → `failIf` → `passIf`/`warnIf`. Outcome is one of:
   - `pass` — advance allowed.
   - `warn` — advance allowed; findings shown; if `approveWarnings: true`, opens an approval card.
   - `fail` — advance denied; findings shown; station re-runs on retry.
7. **Persist result.** Store in `.agentics/runs/<run-id>/gates/<gate-id>.json` (or equivalent platform-side storage). Surface fields from `outcome.surface` on the gate card.
8. **Emit event.** `gate.evaluated` event for downstream consumers (analytics, vibecast timeline).

---

## Runner kinds (extension points — only `cli` defined in this draft)

### `cli` — child process

The only kind enabled by this spec draft. Invokes `command` + `args` directly (no shell). stdout → parsed per `outcome.parse`.

### `http` (future)

```jsonc
"tool": {
  "kind": "http",
  "method": "POST",
  "url": "https://hooks.example.com/lint",
  "headers": { "x-run-id": "{{run.id}}" },
  "body": { "designMd": { "file": "DESIGN.md" } },
  "outcome": { ... }
}
```

### `mcp` (future)

```jsonc
"tool": {
  "kind": "mcp",
  "server": "aspire",
  "method": "doctor",
  "arguments": {},
  "outcome": { ... }
}
```

### `agent` (future, with strict quarantine)

```jsonc
"tool": {
  "kind": "agent",
  "agentId": "ci-reviewer",
  "prompt": "Read DESIGN.md and return JSON {errors:[], warnings:[]}",
  "outcome": { ... }
}
```

This one is **deliberately avoided** for MVP: agents are probabilistic; that defeats the point of a deterministic gate. List here only to reserve the shape.

---

## Execution surface — the deferred question

The open architectural decision is **where** the runner runs. Three candidate locations, with tradeoffs:

### Option A — Station operator (in-broadcast)

The vibecast/station operator invokes the tool inside the same session that just ran the station. Pros: uses the operator's filesystem (no data ship); piggy-backs on existing permissions. Cons: every gate in every project needs the tool installed in the operator; couples gate results to whatever environment the runner happens to be in; no network isolation.

### Option B — Platform sidecar (server-side shell)

The Agentic Live platform pulls the working tree into an ephemeral sandbox and runs the tool server-side. Pros: uniform environment, secrets managed centrally, cacheable across projects. Cons: you now have to host/scale sandboxes and ship code between the operator and the sandbox; more infra.

### Option C — Cloud runner (GitHub Actions / similar)

Offload to an external CI surface via webhook. The platform dispatches a job; the runner POSTs back results. Pros: unbounded scale, already-paid-for compute, isolation by default. Cons: latency, extra hop, error-recovery complexity, requires the project to have a CI account wired.

**Hybrid recommendation (for when the decision is made):** start with **A** for tools that are cheap and dependency-free (design-md lint, tsc, eslint), escalate to **B** for heavy or secrets-dependent tools (dotnet test with a DB), leave **C** for long-running jobs that should not block the broadcast (visual regression, security scan). The gate schema already accommodates this: the executor location is not encoded in the gate itself; the platform routes by tool kind + expected cost.

---

## Security

1. **No shell.** `cli` invocations are argv-only; no templating into a shell string.
2. **Env allowlist.** `tool.env` keys are intersected with a platform allowlist (`platform.envAllowlist`) — unknown keys rejected at schema validation.
3. **Secret injection.** Secrets flow via a separate `tool.secrets: [{ name: "NPM_TOKEN", source: "project.secrets.NPM_TOKEN" }]` block. Registry validates that the named source exists; value never appears in logs/events.
4. **stdout cap.** Runner enforces max stdout/stderr size (e.g., 4 MiB) and truncates with a flag; massive output can't DoS the evaluator.
5. **Timeout as failure.** Timeouts count as `failed` with explicit reason — never left pending.
6. **No network by default.** For `kind: cli`, the executor policy defaults to network-off; tools that need network (like `npx` bootstrapping a package) opt in via `tool.network: "allow"` (requires approval at registration).
7. **Path confinement.** `cwd` must be within `project.workspaceDir`; `tool.stdin.file` must be a relative path inside the same tree.

---

## UX surface (what the gate card shows)

When the tool-backed gate evaluates:

- **Headline**: `pass` ✓ / `warn` ⚠ / `fail` ✗
- **Summary stats**: whatever `outcome.surface.summary` resolves to, rendered as a small key-value table
- **Findings table**: sortable list from `outcome.surface.findings`, with `severity`, `path`, `message` columns (these field names are a convention; the schema doesn't enforce them, but tools following the `@google/design.md` output shape get rendered nicely out of the box)
- **Re-run button** (fail only) — clears cached result and re-dispatches
- **Approve-with-warnings button** (warn only, if gate allows)
- **Raw output drawer** — full stdout/stderr, `exitCode`, `durationMs`, runner version

---

## Migration plan (when execution ships)

1. **Schema-first rollout.** Publish the schema, validate `.agentics/gates.json` against it in the platform. Gates with `tool` blocks but `enabled: false` render as "defined, not executing" in the UI.
2. **Opt-in per gate.** Project owners flip `enabled: true` on individual gates.
3. **Dry-run mode.** A gate can be set to `mode: "dry-run"` — tool runs, result is displayed, but the transition proceeds regardless. Removes the scary-first-time cliff.
4. **Metrics.** Capture `durationMs`, pass/warn/fail counts per gate definition over time. If a gate is flaky (>5% parse errors), auto-disable with a notification.
5. **Retirement path.** Gates can be deleted by clearing their id from `transitions.json`; the gate object lingers in `gates.json` with `enabled: false` until cleaned up. No dangling references — transitions referencing an unknown gate id fail schema validation.

---

## Example — the Design System Lint Gate as used today

Defined in `.agentics/gates.json`:

```json
{
  "id": "design-md-lint-gate",
  "name": "Design System Lint Gate",
  "requiresApproval": false,
  "tool": {
    "kind": "cli",
    "runner": "node",
    "command": "npx",
    "args": ["-y", "@google/design.md@0.x", "lint", "DESIGN.md", "--format", "json"],
    "cwd": "{{project.workspaceDir}}",
    "timeoutSeconds": 60,
    "outcome": {
      "parse": "json",
      "successOn": { "exitCode": 0 },
      "passIf": "summary.errors == 0",
      "warnIf": "summary.warnings > 0"
    }
  },
  "enabled": false
}
```

Wired in `.agentics/transitions.json`:

```json
{ "from": "research", "to": "planning", "condition": "success", "gate": "design-md-lint-gate" }
```

Until `enabled: true` is flipped, the Station 02 orchestrator runs the lint manually and attaches the JSON output to its exit report. A future human reviewing that report sees the same structure the automated gate would eventually evaluate — which means the day execution ships, zero existing projects need to change their behavior.

---

## Non-goals

- **No DSL for tool authoring.** The gate schema calls *existing* tools. If a check needs to be composed, write a tiny script and check it in.
- **No orchestration.** No fan-out, no parallel gate chains. One gate, one tool, one result. Compose by wiring multiple gates on the same transition.
- **No agent-based gates (yet).** The `agent` kind is reserved but deliberately not part of the MVP. Determinism is the point.
- **No long-polling / async gates in v1.** Tool must complete within `timeoutSeconds`. Async workflows (e.g., waiting for a CI build) should use the transition's `condition` field, not gates.

---

## Open questions

1. **Who pins tool versions?** `@google/design.md@0.x` via `npx` is convenient but resolves a different version on every cold run. Options: (a) require exact pins in gate definitions; (b) lock via `.agentics/lockfile.json`; (c) make the platform cache a resolved tarball.
2. **Secrets vault shape.** Project-scoped vs. org-scoped secret sources. Likely already decided elsewhere in the Agentic Live platform — this spec should defer.
3. **Caching invariants.** Hashing `keyInputs` is easy for single-file gates; multi-file gates (e.g., "lint all tests") need a directory-hash convention.
4. **Cost accounting.** Cloud-runner option (C) bills through GitHub or similar. Whether the platform absorbs that or passes it through to the project owner is a product question.
5. **Gate dependencies.** Can gate B reference gate A's output? Probably yes eventually (`{{gates.design-md-lint-gate.result.summary.warnings}}`), but not in MVP.
