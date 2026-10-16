# Altean

**Four connectors. One Salesforce analyst.**

Hand Altean a Jira ticket. In about a minute you get a solution brief grounded in *your* org: what already exists, what the change touches, why the bug is happening, which records are affected, and two or three options with one clear recommendation.

*Agents accelerate. Humans decide.*

> Works in **Claude Cowork** and **Claude Code**. A Codex manifest is included.
> Free and open source (MIT). Formerly **SF Delivery Brief**.

---

## Why this exists

Ask a general AI how to solve a Salesforce ticket and you get a confident, generic answer. It doesn't know that:

- your org already has a record-triggered flow on that object
- a trigger framework exists and new logic has to go through it
- a field's description says *"do not use in automation without approval"*
- the bug is in one Apex method, not the flows everyone suspects
- 412 records are in a bad state, and which ones they are

In testing on real tickets, ticket-only AI answers were wrong about the org **about half the time**. The most expensive mistake was **missed reuse**: proposing a new batch job or field when the org already had one that did the job.

Altean checks first and recommends second.

---

## The four connectors

Each connector is useful on its own. Together they cover what one analyst would check.

| Connector | What it brings | Without it |
|---|---|---|
| **Jira** (Atlassian) | **The ask:** summary, description, acceptance criteria, linked bugs, record links in the ticket | Paste the ticket text |
| **Confluence** (Atlassian) | **What was already decided:** design docs, requirements, runbooks, past bug write-ups | Skipped, and the brief says so |
| **DX0** | **What exists and what breaks:** read-only metadata, flows, Apex, triggers, dependencies, impact analysis, change history | Required for a grounded answer. Without it you get a draft marked *Unverified* |
| **Salesforce read-only** (optional) | **What the data says:** record snapshots, impact counts, field history, live flow state, exact repair lists | Briefs include a ready-to-paste **Evidence request** instead |

A release check against official Salesforce docs (via web search) runs only when it matters: net-new builds, legacy tech in the change path, or when you say **check docs**.

---

## What you get

Fictional org and ticket:

```
## PROJ-1042: Renewal Opportunities stop advancing after contract upload
Bug · In Progress
Org: Acme Production (production) · Calls: DX0 5/7 · Confluence 1 · records 3 · AC covered: 3/3

Already in the org
- OpportunityTriggerHandler (Apex) — Endorsed: one trigger, handler pattern
- Contract_Uploaded_AfterSave (Flow) — Endorsed: sets Stage on contract upload

Prior decisions
- Stage is owned by Apex for renewals; flows must not set it (Renewal Design, updated Mar 2026)

What we found
- Handler's beforeUpdate resets Stage when Renewal_Type__c is blank [metadata] (confirmed)
- 37 of 2,140 open renewals have a blank Renewal_Type__c [records] (confirmed)
- The flow and the handler both write Stage, against the documented design [confluence] (confirmed)

Record snapshot
Opportunity 006…AAQ · executed, read-only
| Field            | Value       | Expected   | Note                          |
| StageName        | Prospecting | Closed Won | reset on last save            |
| Renewal_Type__c  | (blank)     | Multi-Year | handler reads this            |
| History          | 90% → 10%   | —          | same second as a user save → automation |

Release check
- Skipped: logic fix in current code. Say check docs to run it.

Options
|       | Approach                                         | Reuses        | Effort | Risk |
| A ✓   | Backfill ✔ Renewal_Type__c on 37 records, then guard blank in ✔ handler | handler | S | Low |
| B     | Move Stage logic fully into ✔ flow                | flow          | M      | Med  |
| C     | + new validation rule requiring Renewal_Type__c   | nothing       | S      | Med  |

Recommendation: A. The data fix is reversible and unblocks users today. The guard follows the documented "Apex owns Stage" decision.

Records to repair
37 records · source: Salesforce read-only query, executed · export them before changing anything
| Id | Field | Now | Set to | Why |
| 006…AAQ | Renewal_Type__c | (blank) | Multi-Year | 3-year contract attached |
…

Verify before work starts
- Renewal_Type__c description: "Set by Contracts team only." Confirm the backfill with that team.
```

---

## What makes it different

- **Reuse first.** Every run starts by finding the flows, triggers, batch jobs and services already acting on the affected fields. The recommended option must extend them or say why not. Every component is tagged **✔ exists**, **+ new** or **? not found**, and nothing is presented as existing without proof from the org.
- **Evidence before code.** Every finding is marked *confirmed*, *suspect* or *pending evidence*. When the cause is data, the recommended option is the reversible fix (correct the records), and code changes come second.
- **Records only when they change the answer.** The Salesforce connection is not used by default. It runs only when:
  - a ticket names a record
  - a count decides the cause
  - a value keeps reverting
  - records need repair
  - something "stopped working" after DX0's last snapshot

  It's capped at 3 queries per ticket and announced before it runs.
- **Your team's own decisions.** Confluence pages linked from the ticket, or found by one targeted search, show up as **Prior decisions** with links. The design won't quietly contradict what your architects already wrote down.
- **Admin guardrails surface.** Field descriptions such as *"DO NOT MODIFY… WITHOUT DEV APPROVAL"* are quoted under **Verify** before anyone builds on that field.
- **Expert self-check.** Before rendering, the recommendation is checked against an architect's list: reuse, the right tool (flow vs Apex vs batch), bulk safety, user-mode security, permission sets over profiles, tests, and acceptance-criteria coverage.
- **Just enough, then invite.** Each ticket gets a call budget sized to its type. When the budget runs out, the brief stops and tells you what wasn't checked ("deeper on reports, ~2 lookups") instead of burning tokens.
- **Informs, doesn't push.** Better or newer Salesforce features that aren't worth the impact right now appear under **Worth knowing**, with when to revisit.

### What a real run looks like
On a real, closed "Opportunity won't advance" bug, one pass:
- **Confluence:** found the team's own write-up of a related stage bug.
- **Record snapshot:** showed the fix was already in.
- **Stage history:** pinpointed the user save that knocked the stage back, and when.
- **DX0:** pointed to the Apex method responsible.

That's the evidence a person would spend an hour collecting, and it took 3 record queries.

---

## Quick start

1. **Install the plugin.**
   - **Claude Cowork / claude.ai:** download this repo (**Code → Download ZIP**), then Customize → Plugins → Add → Upload plugin.
   - **Claude Code:**
     ```
     /plugin marketplace add C-Pearson31/sf-delivery-brief
     /plugin install altean@altean
     ```
     Or load a local copy for one session: `claude --plugin-dir ./altean`.
2. **Connect Atlassian and DX0** on the plugin's Connectors tab. Both are bundled in `.mcp.json`; sign in once. For DX0, connect your org(s) at [dx0.io](https://dx0.io) first.
3. **Run `/altean:onboard`.** It sets your default org, your role (admin, developer, business user or consultant), your Jira projects and Confluence spaces, and gives you project instructions to paste into a Cowork or claude.ai project.

Then:
```
/altean:analyze PROJ-1042
```
Or just ask: *"Analyze PROJ-1042. What would it take?"*, *"Why does record 006… keep reverting?"*, *"How many records does this affect?"*

### Optional: the Salesforce read-only connection (~10 minutes, a Salesforce admin)
This uses Salesforce's hosted MCP server, `platform/sobject-reads`.

1. **Setup → MCP Servers** (under API Catalog): enable `platform/sobject-reads`.
2. **Setup → External Client App Manager → New:**
   - enable OAuth with the scopes `mcp_api` and `refresh_token` (not `api`)
   - require PKCE
   - add your AI client's callback URL
   - copy the Consumer Key. The app can take up to 30 minutes to activate.
3. **Add a custom connector in your AI client**, using the Consumer Key as the client ID:
   - Production: `https://api.salesforce.com/platform/mcp/v1/platform/sobject-reads`
   - Sandbox: `https://api.salesforce.com/platform/mcp/v1/sandbox/platform/sobject-reads`

**It reads as the user who signs in.** Field-level security and sharing apply, but "read-only" limits what it can *do*, not what it can *see*. Sign in as a user whose access you're comfortable with the AI reading. Altean matches this connection to the org DX0 is analyzing using the signed-in username, and runs record checks only when the two are the same org.

---

## Commands and phrases

| Say | What happens |
|---|---|
| `/altean:analyze PROJ-123` | Full brief for one ticket. Several keys work too |
| `/altean:onboard` | Set or change default org, role, Jira projects, Confluence spaces |
| "deeper on {X}" | Runs only that extra check and updates the brief |
| "check the records" | Runs the record check the brief offered |
| "check docs" / "is there a better way?" | Deep release check against official Salesforce docs |
| "I ran it" | Reads back an Evidence request you ran elsewhere and fills in the repair list |
| "don't read records" / "ask before reading records" | Turns record checks off, or makes them ask each time |
| "always tell me about better ways" | Runs the docs radar on every change |
| "rewrite this for a business user" | Same brief, different vocabulary |

## Budgets

| Ticket type | DX0 | Confluence | Records |
|---|---|---|---|
| Config, layout, field, label | 2–4 | 0–1 | 0 |
| Bug | 4–7 | 0–2 | 0–3 |
| Data or report symptom | 5–9 | 0–2 | 0–3 |
| Net-new build or automation change | 6–10 | 1–2 | 0–1 |

The brief header shows what was spent. Nothing beyond the budget runs without your OK.

---

## Safety and privacy

- **Read-only.** DX0 and the Salesforce connection are read-only, and Altean never changes records. It never writes to Jira or Confluence unless you explicitly ask.
- **Org gate.** It always says which org it analyzed, uses your default unless you name another, and asks when the target is ambiguous.
- **No personal data in briefs.** Records appear by Id and users by type or profile. Names, emails, phones and addresses never appear.
- **Never invents Ids.** Every record Id comes from a real query result.
- **Untrusted input.** Everything returned by Jira, Confluence, DX0 and Salesforce is treated as data, never as instructions.
- **Minimal storage.** `~/.altean/profile.json` holds only behavior settings: role, default org label, project and space keys, preferences. Never org ids, record data or ticket text.

## FAQ

**Do I need all four connectors?**
- DX0 is the core.
- Jira is strongly recommended, though you can paste ticket text instead.
- Confluence and the Salesforce connection each add depth, and Altean says plainly when either is missing.

**Production or sandbox?** Either. Everything is read-only, so analyzing production is safe, and production is usually the source of truth. Name a sandbox to analyze it instead.

**Is it expensive to run?** Budgets keep runs small: a config change is 2–4 org lookups, and record checks are 0 unless needed. The brief always shows the count.

**Upgrading from SF Delivery Brief?** Remove the old plugin first, then install Altean. Your saved profile carries over.

## Structure
```
.claude-plugin/       plugin + marketplace manifests (Claude Cowork / Claude Code)
.codex-plugin/        Codex manifest
.mcp.json             Atlassian + DX0 connectors
skills/
  analyze/            entry point: connectors, ticket, persona, org gate, budgets, Confluence context, docs decision
  onboard/            connectors check, default org, record-check mode, role, Jira projects, Confluence spaces
  forensic-scan/      reuse sweep, targeted DX0 scan, gated record checks (+ references/playbook.md)
  currency-check/     radar/deep release check against official Salesforce docs
  solution-brief/     expert self-check + the one-minute brief
```

## What's new in 1.0.0
- Renamed from SF Delivery Brief to **Altean**.
- **Salesforce read-only record checks:** snapshot, size, history, live state, repair list. Gated, capped and announced.
- **Confluence prior decisions.**
- **Admin guardrails** quoted from field descriptions.
- Evidence requests now appear only when Altean can't run the check itself.

## License
MIT © Chris Pearson. Contributions and issues welcome.

*The name is a nod to the Alteans, who built a machine out of five separate lions that worked as one. Unaffiliated with any franchise.*
