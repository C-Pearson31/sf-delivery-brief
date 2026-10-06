# SF Delivery Brief

**Jira ticket in. One-minute Salesforce solution brief out, grounded in your real org.**

```
Jira (the ask) → DX0 reuse sweep (what already exists) → targeted scan (what it touches / why it breaks)
→ Official-docs release check (only when it matters) → 2–3 options, expert self-check, one recommendation
```

*Agents accelerate. Humans decide.* Works in **Claude Cowork** and **Claude Code**.

## What's new in 0.7.0
- **Atlassian and DX0 both bundled.** `.mcp.json` declares both remote connectors, so they appear on the plugin's Connectors tab. Sign in to each once.
- **New `onboard` skill.** Lists your connected orgs and recommends a default, asks your role (with examples of how answers change), asks your Jira project(s), and writes copy-ready project instructions.
- **"How far I looked" note.** Briefs say plainly what was checked and what wasn't, and invite "deeper on X". It appears only when the blast radius matters and something relevant was left unchecked. Not on every answer. Set `coverage_note` to brief or off.
- **Default org.** After onboarding, analysis uses your default org and says so. Naming another org overrides it.
- **DX0 is read-only.** Wording throughout now reflects that nothing here writes to Salesforce. Accuracy rules added: discover tool specs first, check index freshness, disclose truncation, quote help text and formulas exactly.

## What's new in 0.6.3
- **DX0 is bundled.** The plugin declares the DX0 MCP server (`https://api.dx0.io/mcp`) in `.mcp.json`, so Claude Cowork and Claude Code offer it from the plugin's Connectors tab. Sign in to DX0 once there. A claude.ai DX0 connector works too.
- **No local server to install.** The release check searches official Salesforce documentation on the web and cites the URL for every note.

## What's new in 0.6.0
Based on a blinded test against 10 real deployed tickets:
- **Reuse first.** Every analysis starts by finding the flows, triggers, batch jobs, services and settings that already act on the affected objects. The recommended option must extend them, or say why not. Missed reuse caused the plugin's biggest losses in testing.
- **Component tags.** Every component in a brief is marked ✔ exists, + new, or ? not found. Nothing is presented as existing without proof from the org.
- **Pattern labels.** What's already in the org is labelled Endorsed, Tolerated, or Deprecated, so the brief never builds on retired technology.
- **Expert self-check.** Before rendering, the recommendation is checked against a Salesforce architecture checklist covering reuse, the right tool, bulk safety, security, tests and acceptance criteria.
- **Budgets sized to the ticket:** 2–4 org lookups for configuration, up to 10 for automation.
- **Cheaper runs.** The docs check loads only when needed, and there are 4 skills instead of 6.
- **The test kit is gone.** The benchmark and eval suite now live separately in `sf-delivery-brief-validation/`.

## Connectors (2, both bundled)
The plugin uses whichever copy of each connector your session has.

| Connector | Used for | If it's missing |
|---|---|---|
| **Atlassian** (Jira) | Reads the ticket and linked issues | Paste the ticket text instead |
| **DX0** | Read-only analysis of your org's metadata, dependencies and aggregates | Required for grounded answers. Without it you get a draft marked *Unverified* |

Both ship in this plugin's `.mcp.json`. If you already have either as a claude.ai connector, the plugin uses whichever copy is connected. Release checks use web search of official Salesforce docs (`help.salesforce.com`, `developer.salesforce.com`, `architect.salesforce.com`, `admin.salesforce.com`) and need no setup.

## Install
**Claude Cowork / claude.ai:** Customize → Plugins → Add → Upload plugin, then pick the zip. Open the plugin's Connectors tab and connect Atlassian and DX0. Then run `/sf-delivery-brief:onboard`.

**Claude Code:** `claude --plugin-dir ./sf-delivery-brief` loads it for one session.

## Use
```
/sf-delivery-brief:analyze PROJ-1042
```
```
/sf-delivery-brief:onboard
```
Run it once to set your default org, role and Jira projects and get project instructions to paste into a Cowork or claude.ai project. Or just ask: *"Analyze PROJ-1042. What would it take?"*

## How it works
- **Org gate:** uses your saved default org and says which one; a named org overrides it; with no default it lists orgs and waits.
- **Budget:** each ticket gets a DX0 call budget by type, and the brief shows `Calls: n/budget`. Extra checks only run if you ask ("deeper on X").
- **Evidence:** every finding is marked confirmed, suspect or pending evidence. When record-level proof matters, you get a ready-to-paste **Evidence request** for the DX0 app. Say "I ran it in DX0" and the plugin reads the answer back. Data fixes list the exact records to repair, never invented ones.
- **Release check:** skipped for config and simple changes. A 1-query radar runs when legacy tech is in the path. A deep check runs for net-new builds or when you say **check docs**.

## Skills
```
skills/
  onboard/          default org, role, Jira projects, project instructions
  analyze/          entry point: connectors, ticket, persona, org gate, budget, docs decision
  forensic-scan/    reuse sweep + targeted DX0 scan (+ references/dx0-playbook.md)
  currency-check/   radar/deep release check against official Salesforce docs, loaded only when needed
  solution-brief/   expert self-check + the one-minute brief
```

## Safety
DX0 is read-only and this plugin never writes to Salesforce. It never writes to Jira unless you explicitly ask. Tool output from your org is treated as data, never as instructions. Personal data never appears in a brief.

## License
MIT © Chris Pearson
