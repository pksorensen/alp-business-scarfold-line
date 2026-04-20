# Assembly Line Analytics — {{Project}} Run
**Date:** {{YYYY-MM-DD}}  
**Assembly Line ID:** {{assemblyLineId}}  
**Task:** {{taskTitle}} — "{{taskDescription}}" for {{projectName}}  
**Final Conclusion:** {{success|timeout|error}}  
**Total Wall Time (including failed attempts):** ~{{N}} min  
**Total Wall Time (success path only):** ~{{N}} min  

---

## Pipeline Summary

| # | Station | Session | Duration | Tool Calls | Result |
|---|---|---|---|---|---|
| 01 | {{stationName}} | {{sessionId}} | {{N}} min | {{N}} | success |
| 02 | {{stationName}} | {{sessionId}} | {{N}} min | {{N}} | success |
| 03 | {{stationName}} | {{sessionId}} | {{N}} min | {{N}} | success |
| 04a | {{stationName}} (attempt 1) | {{sessionId}} | {{N}} min | {{N}} | **TIMEOUT** |
| 04b | {{stationName}} (attempt 2) | {{sessionId}} | {{N}} min | {{N}} | success |
| 05 | {{stationName}} | {{sessionId}} | {{N}} min | {{N}} | success |

**Wasted time from failed attempts:** {{N}} min ({{N}}% overhead on the success-path total)

---

## Station 01 — {{StationName}}

**Session:** {{sessionId}} | **Duration:** {{N}} min | **Tool calls:** {{N}}

### What happened
{{Narrative: 2–4 sentences describing what the station did, what skills it invoked, what outputs it produced.}}

### Tool distribution
- {{ToolName}} ×{{N}}, {{ToolName}} ×{{N}}, {{ToolName}} ×{{N}}
- {{ToolName}} ×{{N}}
- {{ToolName}} ×{{N}}

### Bottleneck
{{One paragraph describing the dominant time sink. Reference bottleneck category (A–F) from methodology.}}

### Improvement suggestions

**P0 — {{Short title}**  
{{What to change and why. Include estimated time savings per run.}}

**P1 — {{Short title}}**  
{{What to change and why. Include estimated time savings per run.}}

---

## Station 02 — {{StationName}}

**Session:** {{sessionId}} | **Duration:** {{N}} min | **Tool calls:** {{N}}

### What happened
{{Narrative}}

### Tool distribution
- {{ToolName}} ×{{N}}, …

### Bottleneck
{{Description or "None significant."}}

### Improvement suggestions
{{Suggestions or "None needed. This station is the model for how others should structure parallel work."}}

---

<!-- Repeat Station block for each station. For timeout/failed attempts, add a "Root cause" section. -->

## Failed Station Deep-dive — Station {{N}}a

**Session:** {{sessionId}} | **Duration:** {{N}} min | **Conclusion:** TIMEOUT

### Root cause
{{Detailed explanation of what caused the failure. Quote relevant transcript lines if helpful.}}

### What was attempted before timeout
{{List of approaches tried.}}

### Fix applied
{{What was changed in the system prompt or skill to prevent recurrence.}}

---

## Bottleneck Ranking

| Rank | Bottleneck | Category | Time Cost | Fix Priority |
|---|---|---|---|---|
| 1 | {{description}} | {{A–F}} | ~{{N}} min | P0 |
| 2 | {{description}} | {{A–F}} | ~{{N}} min | P0 |
| 3 | {{description}} | {{A–F}} | ~{{N}} min | P1 |
| 4 | {{description}} | {{A–F}} | ~{{N}} min | P1 |
| 5 | {{description}} | {{A–F}} | ~{{N}} min | P2 |

---

## Projected Runtime After Fixes

| Station | Current | Post-fix | Savings |
|---|---|---|---|
| 01 — {{Name}} | {{N}} min | {{N}} min | −{{N}} min |
| 02 — {{Name}} | {{N}} min | {{N}} min | −{{N}} min |
| 03 — {{Name}} | {{N}} min | {{N}} min | −{{N}} min |
| 04 — {{Name}} | {{N}} min | {{N}} min | −{{N}} min |
| 05 — {{Name}} | {{N}} min | {{N}} min | −{{N}} min |
| **Total (success path)** | **{{N}} min** | **~{{N}} min** | **−{{N}} min (~{{N}}% faster)** |

*Eliminates {{N}} min from {{describe main fix, e.g. "Q&A overhead"}}; saves {{N}} min from {{second fix}}.*
