---
name: solution-brief
description: Renders the final one-minute Salesforce solution brief (request, org, what already exists, prior decisions, findings, record snapshot when a record was read, release notes, 2-3 options with reuse and component tags, recommendation, records to repair, acceptance-criteria coverage, what to verify) pitched to the user's persona, after an expert design self-check. Use as the last step of an analysis, or whenever the user asks to "summarize the options", "give me the short version", "rewrite this for a business user", or "make this scannable".
---

# Solution Brief

One job: turn findings into something a busy delivery person reads in under a minute and can act on, and make sure the recommendation would pass a senior Salesforce architect's review.

## 1. Expert self-check (no calls; fix the options before rendering)
Check the recommended option against each line. If it fails, change the option or name the gap under Verify.

**Fits this org**
- [ ] Extends the existing automation, service, or framework the scan found (reuse-first), or states why not
- [ ] New trigger logic goes through the org's existing trigger framework; no second trigger on an object
- [ ] Reuses existing fields; no near-duplicate field
- [ ] Access follows the org's pattern, preferring permission sets or permission set groups over profile edits
- [ ] Follows the naming seen in the org (prefixes, `Batch_`/`Sched_` styles, flow naming)
- [ ] Leaves middleware and integration-user logic where it lives
- [ ] Respects admin guardrails quoted from field descriptions (e.g. "do not use in automation without approval") and agrees with prior decisions, or says why not

**Correct Salesforce practice**
- [ ] Right tool: before-save flow for same-record field updates; after-save flow or Apex for related records; Apex for complex or high-volume logic; batch or scheduled Apex for annual or bulk jobs. **Never** new Process Builder or Workflow Rules.
- [ ] Bulk-safe: no SOQL or DML in loops; works for 200 records per transaction and batch scope
- [ ] Security: Apex `with sharing` unless justified; `WITH USER_MODE` / `AccessLevel.USER_MODE` in new code (not `WITH SECURITY_ENFORCED`); field-level security via permission sets; Named Credentials, no hard-coded endpoints
- [ ] Errors: validation rules or `addError` with clear messages; fault paths in flows; batch failures reported
- [ ] Tests: bulk (200) and negative cases, run as the real permission persona, assertions on outcomes
- [ ] Current features and the current API version

**Complete**
- [ ] Every acceptance criterion is covered, or the gap is named
- [ ] Blast radius names the real dependents (other flows, reports, integrations, record counts)
- [ ] Every component carries a tag; nothing marked `?` is presented as existing

## 2. Hard rules
- **300 words maximum**, counting everything except SOQL, the record snapshot table and the repair table. Findings ≤ 3 bullets of ≤ 25 words. Option cells ≤ 15 words. Recommendation is 1–2 sentences. Count before sending and cut if over. Never cut the options, the skip line, or a truncation/staleness warning.
- **2 or 3 options**, exactly one marked ✓.
- **Tags on components:** ✔ exists · + new · ? not found.
- **Evidence before code:** if the top cause is pending evidence, or sources disagree, the ✓ option is "confirm with the Evidence request, then apply the reversible fix" (usually a data correction). Code changes come second.
- **Inform, don't push:** Worth knowing never changes the recommendation on its own.
- Plain, direct language. No preamble, no recap of steps. Every claim traces to evidence. No personal data: records appear by Id, users by type or profile.

## 2b. The "How far I looked" note: plain, short, not noisy
Purpose: tell the user what the analysis covered so they can ask for more, without every answer ending the same way.
- **Show it** only when the change's blast radius matters (coverage record significance `high`) **and** something relevant went unchecked, the budget was hit, output was truncated, or the index may be stale.
- **Pure lookups** (a field's help text, a count, "where is X used" answered fully): no note. At most one source line: "Source: {org}, indexed {date}."
- **Low-significance changes** (isolated config, labels, layout): no note unless something was truncated.
- **Follow-ups:** don't repeat it unless coverage changed. Show only the change ("Now also checked reports.").
- **Preference:** `coverage_note: brief` → one line (Checked / Not checked); `off` → omit, except always disclose truncation or a stale index in one line.
- Plain words, no jargon, max 3 "Not checked" items, each tied to why it could matter. Counts must come from the coverage record, never estimated.
- Never state or imply a list is complete when it was truncated or an index type wasn't covered.
- Never present it as a disclaimer. It is an invitation to dig deeper.

## 3. Template

```
## {KEY}: {one-line restatement of the ask}
{Type} · {Status}{ · retrospective: the org may already contain the fix}
Org: {label} ({badge}) · Calls: DX0 {n}/{top}{ · Confluence {c}}{ · records {r}}{ · docs {m}} · AC covered: {x}/{y}

**Already in the org** (what the design builds on)
- {component} ({type}) — {Endorsed|Tolerated|Deprecated}: {what it does, ≤ 12 words}

**Prior decisions** (only if Confluence found something relevant; at most 2)
- {decision or constraint, ≤ 20 words} ([{page title}]({link}), updated {date}{, may be stale})

**What we found**
- {finding} [metadata|aggregates|records|confluence|dx0-history] ({confirmed|suspect|pending evidence})

**Record snapshot** (only when a named record was read)
{Object} {Id} · executed, read-only
| Field | Value | Expected | Note |
(At most 8 rows: only the fields that explain the symptom. Blank shows as "(blank)". Personal fields are never shown.)

**Release check**
- {note → adopt/avoid} ({source URL})
  — or, when skipped: Skipped: {reason}. Say **check docs** to run it.

**Worth knowing** (only if any; at most 1)
- {better way} — not recommended now: {reason}. Revisit when {trigger}.

**Options**
| | Approach | Reuses | Effort | Risk | Why / why not |
|---|---|---|---|---|---|
| **A ✓** | {approach with tagged components, e.g. extend ✔ Batch_X; + new field Y__c} | {what it extends, or "nothing: {reason}"} | S/M/L | Low/Med/High | ... |
| B | ... | ... | ... | ... | ... |

**Recommendation:** A, because {reason tied to a finding and the reuse decision}.

**Records to repair** (only when record evidence is in hand)
{n} records · source: {Salesforce read-only query, executed | DX0 app conversation | sf CLI} · export them before changing anything
| Id | Field | Now | Set to | Why |
(Up to 25 rows; more than 25: show 10 plus the total, and offer "full repair CSV" in the How far I looked note.)

**Verify before work starts** (at most 3: the ones that could change the recommendation; any `?` component first; then any admin guardrail, quoted; then any uncovered acceptance criterion)
- {check}

**Evidence request** (only when record proof would change the recommendation **and** it couldn't be run here: no Salesforce read-only connection, a different org, or record reads off)
> In {org}: {check} `{SOQL}`. Confirm {suspect} and list the records to fix (Id, field, target value).
Run it with your Salesforce read-only connection for {org}, the DX0 app, or the `sf` CLI, then say "I ran it".

**How far I looked** (only when the rules in section 2b say so)
Checked: {plain list with counts}. Not checked: {1–3 items that could matter}. Org data as of {index date, if known}.
Want more? Say "deeper on {item}" (~n lookups).
```

Leave out empty sections. Options, Recommendation and **Already in the org** always appear. If nothing relevant exists, write "Nothing found on {object} for this. Searched {terms}."

## 4. Persona tuning
The structure stays the same; vocabulary and the "Why / why not" column change.
- **developer**: classes, flows, triggers, governor limits, test coverage, deployment order
- **admin**: setup path, Flow vs configuration, permissions, sandbox testing
- **business**: outcome, time and risk in plain words; API names only in a closing "For your developer:" line
- **consultant**: options as client trade-offs (cost, risk, time-to-value, maintainability)
- **neutral**: balanced

## 5. Effort and risk
- **Effort:** S = configuration or a small code edit · M = a sprint-sized story · L = several stories, or needs design. Relative, not a commitment.
- **Risk:** dependents, production exposure, external consumers, critical DX0 signals.

End with **Agents accelerate. Humans decide.** only the first time in a session.
