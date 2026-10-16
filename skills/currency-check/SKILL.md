---
name: currency-check
description: Runs a radar (1 query) or deep (up to 2 queries) Salesforce release-currency check against official Salesforce documentation via web search, to surface better, newer, or retiring ways to solve the problem without pushing a change. Loaded by `analyze` only when a docs check is needed, or when the user asks "check docs", "is there a better way", "is this still the right way", "did Salesforce ship something for this", or "any release changes we should know about".
---

# Currency Check

One job: the user should never find out later that a better, newer, or safer way existed and nobody mentioned it. **Informing is not recommending.** In enterprise orgs the right call is often to leave things as they are, but the user still deserves to know.

`analyze` decides the level. Don't run this skill to decide whether to skip; that is already done.

| Level | Queries | Purpose |
|---|---|---|
| **radar** | 1 | "Is there a better or newer way, or a retirement notice, for this one area?" Results go to **Worth knowing**. |
| **deep** | ≤ 2 | Can change the options and the recommendation. |

Docs queries have their own allowance, separate from the DX0 budget. Check each feature area at most once per session and reuse the answer.

## Ask narrow questions
- Radar: "Current recommended approach, and any retirement notice, for {feature} as of the latest release?"
- Deep (pick the 1–2 that matter): a native feature for {X}; deprecations for {feature or API}; default-behavior changes for {user mode / sharing / API version / Flow element}; limits for {feature} in {context}.

## Source
Official Salesforce documentation only: `help.salesforce.com`, `developer.salesforce.com`, `architect.salesforce.com`, `admin.salesforce.com`, and the release notes for the current release. Use web search, then fetch the best official page when the snippet isn't enough. Cite the URL for every note. If no official source supports a point, leave the point out; never answer from memory.

## Classify every result
- **adopt**: changes the options now (deep only), and only when its benefit outweighs the blast radius the scan found.
- **avoid**: a retirement or breaking change the current options would walk into. Always surface it.
- **worth knowing**: a better way that isn't recommended now. State why not now (impact, scope, timing) and when to revisit.

With a high blast radius (many dependents, production-critical automation, external consumers), prefer **worth knowing** over **adopt**.

## Return
At most 2 notes for deep and 1 for radar. Each has What, Class, Why not now (for worth knowing), and Source (URL). If the radar finds nothing, return nothing.
