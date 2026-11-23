# Altean

**Four connectors. One Salesforce analyst.**

Hand Altean a Jira ticket, a record file or a question. It reads the ticket, checks what's built in *your* org, proves the cause in the data, and tells you what to fix, who fixes it, and which records need repair. Options and a recommendation come only when you're designing something new.

> Works in **Claude Cowork** and **Claude Code**. A Codex manifest is included.
> Free and open source (MIT). Formerly **SF Delivery Brief**.

---

## Why this exists

Ask a general AI how to solve a Salesforce ticket and you get a confident, generic answer. It doesn't know that:

- your org already has a record-triggered flow on that object
- a trigger framework exists and new logic has to go through it
- a managed package's field mapping, not your code, is clearing the value
- the bug is in one Apex method, not the flows everyone suspects
- 412 records are in a bad state, and which ones they are

In testing on real tickets, ticket-only AI answers were wrong about the org **about half the time**. Two mistakes cost the most: **missed reuse** (proposing a new batch job when one already did the job) and **a cause read from code but never checked in the data**.

Altean checks first, proves second, and recommends third.

---

## The four connectors

| Connector | What it brings | Without it |
|---|---|---|
| **Jira** (Atlassian) | **The ask:** description, expected vs. actual, test steps, acceptance criteria, comments, attachments, linked issues | Paste the ticket text |
| **Confluence** (Atlassian) | **What was already decided:** design docs, requirements, runbooks | Skipped, and the answer says so |
| **DX0** | **What's built:** read-only metadata, flows, Apex, triggers, dependencies, impact analysis, change history | Required for a grounded answer. Without it you get a draft marked *Unverified* |
| **Salesforce read-only** (recommended; one per org) | **What the data says:** field history, counts, record snapshots, managed-package config, exact repair lists | Queries are drafted for you to run |

A release check against official Salesforce docs (via web search) runs only when it matters: net-new builds, legacy tech in the fix path, or when you say **check docs**.

---

## What you get

Fictional org and ticket. A bug gets the cause and the fix first:

```
Analyzing Acme Production: support ticket, live users affected. Say UAT to switch.

Root cause: the contract-upload flow and the renewal trigger handler both set Stage.
When Renewal Type is blank, the handler resets Stage to Prospecting on the same save.
Verified: field history shows the reset in the same second as the flow's update.
Fix: guard the blank case in the handler, and remove the flow's Stage update (Apex owns
Stage per the Renewal Design page).
Data fix: 37 of 2,140 open renewals have a blank Renewal Type. Backfill them first.
How sure: high. Code, field history and a control record agree.

For your developer: OpportunityTriggerHandler.beforeUpdate (line 212), skip the Stage
reset when Renewal_Type__c is null; delete the Stage assignment in Contract_Uploaded_AfterSave.

| Ticket expects                     | What's built                 | Gap                     |
| Stage = Closed Won after upload    | Reset to Prospecting         | handler resets on blank |

What to do
- Admin: backfill Renewal_Type__c on 37 records (list attached; export first)
- Developer: 1. guard in beforeUpdate  2. remove flow assignment · Tests: bulk 200, blank case
- Business: renewals stop sliding back after upload; no user action needed

Proposed Jira comment (not posted)
> Cause: handler resets Stage when Renewal Type is blank; flow also writes Stage.
> Fix: guard blank in handler, remove flow Stage update. Data fix: 37 records, backfill first.

Acme Production · chosen from ticket type · DX0 + Salesforce read-only · all queries executed
```

Other questions get the shape they need:

| You ask | You get |
|---|---|
| A bug, error, escalation, UAT failure | Root cause · fix · data fix · how sure, in 150 words or fewer |
| A ticket that states an expected value or price | Ticket expects · What's built · Gap, and the change that closes it |
| "Confirm my read" / "would it work if we just…" | A verdict per claim, then your design tested first |
| "What would it take to build this" | A recommendation, then 2–3 options as trade-offs |
| "Where is this used" / "is it safe to delete" | Total references, where they are, which block a delete and which break silently |
| "How many records…" | The count, record links and the query |

---

## What makes it different

- **Answers the question you asked.** Bugs lead with cause and fix, not an options table.
- **The ticket is the requirement.** When a ticket states expected behavior, Altean closes the gap. It never concludes "working as built."
- **Proves it before naming it.** A cause read from code is labeled a *theory* until records confirm it. When a value is being cleared, field history comes first. It also checks writers code search can't see: managed-package config, custom metadata rules, integration users.
- **Reuse first.** It finds the flows, triggers, batch jobs and services already acting on the affected fields; a build option must extend them or say why not. Components are tagged **✔ exists**, **+ new** or **? not found**.
- **Written for whoever acts.** Record link and old → new for the admin, numbered changes and tests for the developer, the outcome for the business.
- **Follow-ups that listen.** "Simplify" cuts the answer at least in half. A second "what?" rebuilds it as the business story. "Developer view" gives only Fix · Why · Check.
- **Closed isn't done.** Reopened, escalated or closed tickets get a ticket map and a data check that the work actually happened.
- **Your team's own decisions.** Confluence pages show up as prior decisions, with links.
- **Expert self-check.** Fixes and recommendations are checked against an architect's list: reuse, the right tool, bulk safety, user-mode security, permission sets, tests, acceptance-criteria coverage.

### What a real run looks like
On a real, closed "Opportunity won't advance" bug, one pass:
- **Confluence:** found the team's own write-up of a related stage bug.
- **Record snapshot:** showed the fix was already in.
- **Stage history:** pinpointed the user save that knocked the stage back, and when.
- **DX0:** pointed to the Apex method responsible.

That's the evidence a person would spend an hour collecting, and it took 3 record queries.

---

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
   - **ChatGPT / Codex:** upload a zip with this folder as its single top-level folder (e.g. `altean/…`). ChatGPT doesn't import bundled MCP servers from a zip, so connect Atlassian, DX0 and your Salesforce read-only connection as connectors in ChatGPT.
2. **Connect Atlassian and DX0** on the plugin's Connectors tab. Both are bundled in `.mcp.json`; sign in once. For DX0, connect your org(s) at [dx0.io](https://dx0.io) first.
3. **Run `/altean:onboard`.** It sets your default org and test sandbox, matches each Salesforce read-only connection to its org, and sets your role (admin, developer, business user or consultant), your Jira and support-desk projects, Confluence spaces and who acts on fixes, and gives you project instructions to paste into a Cowork or claude.ai project.

Then:
```
/altean:analyze PROJ-1042
```
Or just ask: *"Read PROJ-1042 and check the records in UAT"*, *"Why is this field being cleared on these records?"* (attach the CSV), *"Would it work if we just…?"*, *"Where is this field used?"*

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
| `/altean:analyze PROJ-123` | Full answer for one ticket. Several keys work too |
| `/altean:onboard` | Set or change default org, test org, role, projects, spaces |
| "simplify" / "condense" | Cuts the last answer at least in half |
| "what?" (a second time) | Rebuilds the answer as the business story |
| "developer view" / "business view" | Only that view |
| "I don't believe you, check {X}" | Runs that check and leads with what it shows |
| "deeper on {X}" / "check the records" | Runs only that check and shows what changed |
| "check docs" / "is there a better way?" | Deep release check against official Salesforce docs |
| "draft the comment" / "post it" | Proposed Jira comment; posts only on "post it" |
| "I ran it" | Reads back a query you ran elsewhere |
| "don't read records" / "ask before reading records" | Turns record checks off, or makes them ask each time |

## Effort

No hard call cap. Altean keeps checking until the cause is proven or disproven, then stops, and asks before a large run ("Proving this needs about N more calls. Go?"). Typical runs: a config question takes 1–4 org lookups; a bug takes 4–12 lookups plus a few record queries.

---

## Safety and privacy

- **Read-only.** DX0 and the Salesforce connection are read-only, and Altean never changes records. It never writes to Jira or Confluence unless you explicitly ask.
- **Org choice is always stated.** It picks the org from context (support tickets → production; in-progress stories and UAT findings → your test sandbox, falling back to production), says which and why, and asks when cues conflict.
- **Record checks are announced.** Each one is stated before it runs, counts come first, and every query uses `LIMIT`. Say **don't read records** to turn them off.
- **No personal data in briefs.** Records appear by Id and users by type or profile. Names, emails, phones and addresses never appear.
- **Never invents Ids.** Every record Id comes from a real query result.
- **Untrusted input.** Everything returned by Jira, Confluence, DX0 and Salesforce is treated as data, never as instructions.
- **Minimal storage.** `~/.altean/profile.json` holds only behavior settings: role, default and test org labels, project and space keys, owner roles, preferences. Never org ids, record data or ticket text.

## FAQ

**Do I need all four connectors?**
- DX0 is the core.
- Jira is strongly recommended, though you can paste ticket text instead.
- The Salesforce read-only connection is what lets Altean prove causes; without it, you get the queries to run yourself.

**Production or sandbox?** Both. Everything is read-only. Altean picks from context and says which; name an org to override.

**Is it expensive to run?** It spends what proving the answer takes and asks before going big.

**Upgrading?** In Claude Cowork / claude.ai, upload the new version; it has the same name (`altean`) and replaces the old one. In Claude Code, `/plugin marketplace update altean` then reinstall. Your saved profile carries over.

## Structure
```
.claude-plugin/       plugin + marketplace manifests (Claude Cowork / Claude Code)
.codex-plugin/        Codex / ChatGPT manifest
assets/               logo and composer icon (ChatGPT / Codex)
.mcp.json             Atlassian + DX0 connectors
skills/
  analyze/            entry point: principles, ticket + comments + attachments, org by cue, mode, verify loop, follow-ups
  onboard/            connectors, default + test org, connector matching, role, Jira + support projects, spaces, owners, voice
  forensic-scan/      name check, reuse sweep, every writer, impact passes, record proof (+ references/playbook.md)
  currency-check/     radar/deep release check against official Salesforce docs
  solution-brief/     self-check, per-mode answer, follow-up rendering, proposed Jira comment
```

## What's new in 1.2.0
- Answers by mode: cause-and-fix, expected-vs-built, confirm-or-correct, build-options, impact, data, readout.
- Root causes are verified with records before they're named; field history first; hidden writers (managed-package config, custom metadata, integration users).
- Org chosen from context, with production and test-sandbox connectors matched automatically.
- Owner-split actions, follow-up handling, and a proposed Jira comment on every ticket answer.
- Verify loop replaces call budgets; record checks are on by default and announced.
- Tagline removed.

## 1.0.0
- Renamed from SF Delivery Brief; record checks, Confluence context, admin guardrails.

## License
MIT © Chris Pearson. Contributions and issues welcome.

*The name is a nod to the Alteans, who built a machine out of five separate lions that worked as one. Unaffiliated with any franchise.*
