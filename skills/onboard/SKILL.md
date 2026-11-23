---
name: onboard
description: One-time personalization for Altean. Checks the connectors, lists the user's connected Salesforce orgs and recommends a production default plus a test sandbox, matches each Salesforce read-only connector to its org, asks the user's role, their Jira project(s) and support-desk project(s), Confluence space(s), who acts on fixes, and whether they have a voice skill for comments, then writes short copy-ready project instructions. Use when the user says "set me up", "onboard", "personalize", "write my project instructions", "change my role", "change my default org", "change my test org", "change my Jira project", "change my Confluence space", or right after installing the plugin.
---

# Onboard

One short conversation that makes every later answer fit this person. Ask one question at a time. Never write to Salesforce, Jira or Confluence.

## 1. Check connectors (no calls)
One line each, ✔ or ✗: **Jira** and **Confluence** (Atlassian), **DX0**, and **Salesforce read-only** (one per org; often deferred: run a tool search for `getUserInfo soqlQuery` before marking it ✗). Continue with what's available.

## 2. Orgs
Call `dx0_get_connections` (`readyOnly: true`) and list `{n}) **{label}** ({badge}) · workspace {workspace}`.
- **Default (production):** with exactly one production org, recommend it: DX0 and the record connectors are read-only, and production is the source of truth. Otherwise ask.
- **Test org:** "Which sandbox do you test stories in? I'll use it for in-progress stories, UAT findings and test steps, and fall back to the default when it isn't connected." (or **none**)
- **Opt-in only:** orgs for another client or person: "used only when you name it".

**Record connectors:** call `getUserInfo` on each Salesforce read-only connector. Match its `username` to a DX0 connection's `sublabel` and say "**{connector}** reads **{DX0 label}**." Ask about any that don't match. Then: "Record checks: **auto** (whenever they confirm a cause, size impact or list records to fix; announced first), **ask** each time, or **off**?" Default `auto`.

## 3. Role
"What's your role: **admin**, **developer**, **business user**, or **consultant**?" One example of how it changes answers: for "add a required field on Opportunity", an admin hears "layout + validation rule, test in sandbox"; a developer hears "check triggers and Flows on Opportunity, update tests"; a business user hears "small change, low risk, 1 day".

## 4. Jira
- "Which Jira project key(s) hold your stories and bugs?"
- "Do any projects hold support-desk tickets that are always about production? (e.g. a service-desk key)" Save as `support_projects`.
If unsure and `searchJiraIssuesUsingJql` is available, offer one read-only search (`assignee = currentUser() ORDER BY updated DESC`, max 10) and propose keys. Save only after a yes.

## 5. Confluence (optional)
"Which space key(s) hold designs, requirements or runbooks? (or **skip** to search all)"

## 6. Who acts, and your voice
- "Who usually acts on fixes? e.g. support analyst or admin for record changes, developers for code." Save roles (names optional).
- "Do you have a skill that writes in your voice for Jira comments? (name it, or **no**)" Save as `voice_skill`.

## 7. Output
1. Summarize the choices.
2. Save `~/.altean/profile.json` if you can write files:
   `{ "persona": "...", "default_org": "...", "test_org": "...|null", "opt_in_only_orgs": [], "jira_projects": [], "support_projects": [], "confluence_spaces": [], "owners": {...}, "voice_skill": "...|null", "coverage_note": "normal", "docs_mode": "standard", "record_reads": "auto", "use_history": true }`
   Labels only: no org ids, client data or ticket text. If you can't write files, say so; the project instructions carry the settings.
3. Give a copy-ready **project instructions** block. The plugin holds the method, so keep this to who and where:

```
You are my Salesforce analyst and delivery partner. My role: {persona}.
For any ticket key, record file, or Salesforce bug, data, impact or build question, run /altean:analyze, and keep using it for follow-ups (simplify, what?, developer view, check X, draft/post comment).
Orgs: default {default_org} (production). Test org: {test_org}; use it for in-progress stories, UAT findings and test steps, and fall back to {default_org} when it isn't connected. {sandbox} only for "what will my branch look like". {opt-in orgs} only when I name them. Always say which org and why.
Record connectors: {connector} reads {org}; {connector} reads {org}. Never use one org's connector for another.
Jira: stories and bugs in {projects}; support-desk tickets in {support_projects} (always production). Never create, edit, comment on or transition issues unless I ask.
Confluence: check {spaces} for prior decisions before designing.
Who acts: {owners}. Voice for comments: {voice_skill}.
Org facts and traps: {add any known data traps, e.g. text-formula booleans, hygiene filters}.
```
4. Close with: "Say **change my default org**, **test org**, **role**, **Jira project** or **Confluence space** any time, or run `/altean:onboard` again."

## Rules
- Role and orgs come from the user's answers, never from history alone. Propose, then save after a yes.
- Keep it to about eight short questions. Re-running updates only what changes.
