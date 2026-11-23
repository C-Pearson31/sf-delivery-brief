---
name: solution-brief
description: "Final render step of the Altean analysis, loaded by `analyze` (including on follow-ups like simplify, what?, developer view, draft or post the comment). Applies an expert self-check, then renders the answer for the chosen mode — cause-and-fix, expected-vs-built, confirm-or-correct, build-options, impact, data or readout — with a 150-word plain top section, a \"For your developer:\" line, owner-split actions and a proposed Jira comment marked not posted. Do not run it directly from a user request; route through `analyze` so org and mode are set first."
---

# Solution Brief

One job: the user reads the top once and knows what's wrong (or what to build), why, and what to do next, without knowing the codebase. Everything below the top is evidence they can skip.

## 1. Expert self-check (no calls; fix the answer before rendering)
Check the fix or recommendation against each line. If it fails, change it or put the gap under **Before you build**.
- Closes every gap in the expected-vs-built rows; covers every acceptance criterion, or names the gap
- Extends the existing automation or framework the scan found, or states why not; no second trigger on an object; no near-duplicate field
- Right tool: before-save flow for same-record updates; after-save flow or Apex for related records; Apex for complex or high-volume logic; batch/scheduled Apex for bulk jobs; **never** new Process Builder or Workflow Rules
- Bulk-safe (no SOQL/DML in loops; 200 records); `with sharing` and user mode in new code; permission sets over profile edits; Named Credentials
- Accounts for every writer found, including managed-package config, custom metadata rules and integration users
- Respects quoted admin guardrails and prior decisions; stays inside the user's stated scope
- Tests: bulk and negative cases, run as the real persona
- Nothing tagged `?` presented as existing; no theory presented as verified

## 2. Hard rules
- **The top section is 150 words or fewer**, plain language, no API names. API names go in one **For your developer:** line right after it.
- Lead with the answer. No preamble, no recap of steps, no restating the ticket at length.
- **Options only in build-options mode**, or when a real design choice exists inside the user's scope. A debug log, reproduction or record check is a check, not an option; it goes under **Before you build**.
- Tables for comparisons, expected-vs-built and ticket maps. Short bullets otherwise.
- Bold key terms. 🎯 on at most two critical callouts. For multi-part answers, open with `> ⏯️ {one-line coverage summary}`.
- Records the user or an admin must open or change appear as full Lightning links (`{instance_url}/lightning/r/{Object}/{Id}/view`); if the instance URL is unknown, give the relative path and say so.
- Dollars as currency ($1,050.00). Job, account, order and spec numbers as text (no commas, no rounding). Ids exact.
- No personal data. Records by Id or link, people by role.
- **No internal labels in output:** no Endorsed/Tolerated/Deprecated, no `[metadata]`-style source tags, no "Calls: n/m" headers, no mode names.
- **No tagline, slogan or sign-off.**
- End every data answer with one line: org, how it was chosen, tools used, executed or drafted. Add copy-ready SOQL when it helps the user verify.
- **"Not checked"** line only when the cause is still a theory, you stopped at the ask-first threshold, output truncated, or the index may be stale. Max 3 items, each with why it could matter.

## 3. Templates (leave out empty sections)

### cause-and-fix (bugs, errors, escalations, UAT failures)
```
**Root cause:** {one or two plain sentences}. {If theory: "Theory, not proven. The one check that settles it: {check}."}
**Fix:** {what changes, in plain words}
**Data fix:** {none: records are fine | {n} records need {change}; method}
**How sure:** {high/medium/low and why, one line}

For your developer: {class.method (line) / flow element; what to change}

**Expected vs built** (when the ticket states a target)
| Ticket expects | What's built | Gap |

**Evidence**
| Check | Result |
(field history, record counts "n of total", control check, code lines read)

**What to do** (one block per owner who acts; omit owners with nothing to do)
- Support/admin: {record link} · {field}: {old} → {new}
- Developer: 1. {change} 2. {change} · Tests: {what to update}
- Business/PO: {outcome}

**Before you build** (≤ 3; only checks that could change the fix)

**Proposed Jira comment (not posted)** — see §6

{org · how chosen · tools · executed/drafted}
```
If a check disproved an earlier answer, the first line is "This corrects what I said earlier: …".

### expected-vs-built (standalone, e.g. a pricing or value ticket)
Lead with one sentence: "The ticket expects {X}; the org does {Y}; change {Z} to match." Then the table, the owner actions, the Jira comment.

### confirm-or-correct
```
**Your read**
| Your claim | Verdict | Why (evidence) |
(✅ correct / ⚠️ partly / ❌ not as written)

**Your design, tested as option A**
| Requirement | Met? | Note |
Then: keep A as is | adjust A by {change} | only if A fails a requirement, the alternative.
```
Never replace the user's design before testing it.

### build-options
```
**Recommendation:** {A, in one or two sentences, tied to a finding and the reuse decision}

| | Approach | Reuses | Effort | Risk | Maintainability | Why / why not |
| **A ✓** | {tagged components: extend ✔ X; + new Y} | ... | S/M/L | Low/Med/High | ... | ... |
| B | ... |

**Already in the org** (≤ 5) · **Prior decisions** (≤ 2, with links) · **Release check** (only if run)
**Build detail** (only for the recommended option, compact): setup path · Flow vs config · permissions · classes/triggers · governor limits · tests · deployment order
```
Persona tunes the trade-off column: developer (code, limits, tests, deployment), admin (setup, Flow vs config, permissions, sandbox), business (outcome, time, risk), consultant (cost, risk, time-to-value, maintainability).

### impact
Total reference count in the first line, then `Where | Count | Which` (≤ 5 names per row, then "+N more"), then "Blocks a delete:" and "Breaks silently:", then unindexed types not covered.

### data
The answer (counts, distributions) in the first line, record links for anything to open, then the copy-ready SOQL and the footer line.

### readout (then STOP)
What's expected · Current state · Conflicts · Open questions · Proposed next scan. End with "Say **proceed** to run it."

### Ticket map (when a sweep ran: reopen, escalation, closed ticket, several tickets)
| Ticket | Status | What it was | What it means here |
Never treat Closed/Done as proof the work happened; say what the data shows.

## 4. Follow-up rendering
- **"Simplify", "condense", "brief", "shorter":** cut the previous answer by at least half. Keep only what the user asked for. Never re-add caveats, options, monitoring ideas or questions that were cut.
- **Second "what?", "doesn't make sense", "I don't understand":** the frame was wrong, not the wording. Rebuild as the business story: what was asked, what was built, what was missed, what fixes it. Use the ticket history and a short ticket map when several tickets are involved. No API names except one "For your developer:" line.
- **"Developer view" / "what the developer needs to fix":** per issue only **Fix** · **Why** (≤ 2 sentences) · **Check**. No options, business questions, monitoring or history. Config work and code work become separate items when owners differ.
- **"Business view" / "admin view":** only that view.
- **Deeper / check docs / new check:** show only what changed, or one line saying nothing changed. Never re-render the full brief.
- **"Give me the full text" / any revision of something to paste:** re-render the whole thing, ready to paste. Never send a diff.

## 5. Effort and risk
Effort: S = config or a small code edit · M = a sprint-sized story · L = several stories or needs design. Risk: dependents, production exposure, external consumers.

## 6. Proposed Jira comment
End every bug or ticket answer with one, marked **(not posted)**, written in the user's voice (use the profile's `voice_skill` if set and installed).
- **For a developer:** cause and fix as bullets.
- **For a support or service-desk requester:** cause in plain words, **Data fix:** (none, or count and method), and one check to run before building.
- Add one "Separate question for later:" only when something is genuinely unresolved and the ticket doesn't already answer it.
- **Before posting** on a support-desk ticket, or a ticket linked to one, ask whether it should be **public or internal**.
- Post only after the user says "post it". Then report the comment Id and its visibility.
