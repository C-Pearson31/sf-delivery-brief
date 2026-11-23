# Connector Playbook (DX0 + Salesforce read-only)

Tool costs, argument names, and gotchas, taken from live probing. The catalog is served live and can change. If an id fails, call `dx0_discover({ tools: [id] })` for the current schema.

## Calling pattern
DX0 is read-only: no tool here writes to Salesforce.
- `dx0_get_connections({ readyOnly: true })`: once per session.
- `dx0_discover({ connectionIds })` for the catalog, then `dx0_discover({ tools: [ids] })` for the full spec of each tool before its first execution in a session. Discovery calls don't count toward the ask-first threshold. The shapes below are hints; the live spec wins.
- Never guess connection ids, tool schemas, object names, or field names.
- Check `get_index_status` for recent-change questions, unexpected misses, or before searching a folder that may not be indexed. An unindexed type is never proof of absence.
- Large results are truncated. Say so, and never call a truncated list complete.
- `dx0_execute({ tool, connectionId, arguments })`. Workspace-level tools take `workspaceId` instead.

## Cost guide (prefer light)

| Tool | Cost | Use for | Arguments |
|---|---|---|---|
| `search_metadata` (`files_with_match_only`) | light | Locating anything | `query` or `queries[]`, `path`, `glob`, `regex` |
| `search_metadata` reuse sweep (`files_with_match_only: true`, `regex: true`, `query: "Field_A__c\|Field_B__c"`) | light | The required "what already exists" step | no `path`; sort the file list by folder instead of rerunning |
| `search_metadata` (content mode, one file) | light | Seeing a match in context | add `glob: "<File>.cls"` |
| `read_metadata_file` | light–med | Confirming a suspect | `file_path`, `start_line`, `end_line` (snake_case) |
| `search_impact` | med | "What breaks / where used" for ONE component; also returns free record counts | `name` (`Object.Field`), `type`, `max_files: 5` |
| `get_change_log` | light | "Stopped working": recent snapshot commits | `limit` |
| `signal_multipleTriggers` | med | Trigger order conflicts | none |
| `list_conversations` / `get_conversation` | light / med | Past DX0 investigations; reading back a DX0 app answer | `connectionId` / `conversationId` |
| `signal_objectContext` | **heavy** | Only when options touch validation rules, record types or OWD | `objectName` |
| `signal_dataDictionary` | **heavy** | Only when the user asks for the field list | `objectName` |
| `signal_changeHistory` | **heavy** (personal names) | Only when audit-trail detail is the question | none |
| Content-mode `search_metadata` over a whole folder | **heavy** | Avoid, especially `Flow/` | |
| `search_impact` on a hugely referenced field (e.g. `Account.Name`) | **fails** | Never. Exceeds the 5MB output limit. | |

DX0 guides (`dx0_get_skill`): `ImpactAnalysis` for where-used/rename/delete, `OrgConfigSources` for what a status or setting does, `ApexTriggers`/`ApexTesting` before proposing that code, `CPQ` when `SBQQ__` is in scope; `SOQL`/`SOSL`/`Formulas` only when a query or formula misbehaves. Once per topic per session.

## Record data
- **DX0 MCP:** no free-form SOQL. DX0's own SOQL guide says its MCP tools don't run it. What it does return for free:
  - **aggregates:** `search_impact.utilization` gives total and populated counts, plus counts per record type
  - **live signals:** for example users, integrations, OAuth tokens, change history
- **Salesforce read-only connection** (Salesforce hosted MCP, `platform/sobject-reads`): runs real SOQL. This is the record path when it's connected and reads the same org (see below).
- **Fallbacks:** the DX0 app (its agent runs read-only SOQL; read the answer back with `list_conversations` + `get_conversation`) or an authenticated `sf` CLI.

# Salesforce read-only connection

Tool names come from the Salesforce hosted MCP and may carry a server prefix. Tested live.

| Tool | Use for | Notes |
|---|---|---|
| `getUserInfo` | Org match, once per connector per session (free) | Returns `username` and admin-set defaults; may include the instance URL (use it for Lightning links; if absent, give relative paths). Match `username` to the DX0 connection's `sublabel`. |
| `soqlQuery(q)` | Snapshot, size, history, repair list, live state | Always `WHERE` + `LIMIT`. Max 50,000 rows per transaction. |
| `find(q)` | A record number or name with no Id | SOSL; max 2,000 rows; `RETURNING {Object}(Id) LIMIT 5`. |
| `getObjectSchema(objects)` | Field API names, picklist values, descriptions, help text, admin data-quality guidance | One or more objects, comma-separated. Output is large: grep it rather than reading it whole. With no argument it returns the list of queryable objects. Call it on the same connector before any SOQL on that object. |
| `getRelatedRecords(sobject-name, id, relationship-path)` | Children of one record | Use the relationship name (e.g. `Opportunities`, `Contacts`, `{Custom}__r`). |
| `listRecentSobjectRecords(sobject-name)` | Rarely; "the record I was just looking at" | Up to 2,000 rows. |

## Traps (each one failed in a live test)
- History objects: `CreatedBy` is polymorphic, so `CreatedBy.UserType` fails. Select `CreatedById`, then query `User` for `UserType` and `Profile.Name`.
- Opportunity stage changes are in `OpportunityHistory` (always on), **not** `OpportunityFieldHistory`, which returned 0 rows for `StageName` on a record whose stage had changed 4 times.
- `SetupAuditTrail.Section` can't be filtered. Filter on `CreatedDate`, then read `Section` in the results.
- `FlowDefinitionView.LastModifiedDate` is often null (always for managed flows). Compare `ActiveVersionId` with `LatestVersionId` instead.
- The connection reads **as the signed-in user**. Field-level security and sharing apply, so a field or record that's missing may be hidden from that user rather than absent. Say "not visible to the connected user".
- "Read-only" describes the tools, not the reach: an admin sign-in can read almost everything. Query only what the check needs.
- Production and sandbox are separate connections (sandbox URLs add `/sandbox/` after `v1`). Never assume the connected org.

## Argument traps (each one cost a wasted call in live runs)
- `search_metadata.regex` is a **boolean** (`regex: true`), not a pattern. The pattern goes in `query`.
- There are no inline regex flags (`(?i)`). Searches are case-insensitive by default.
- `path` must be a **folder** (`ApexClass`, `Flow`). To target one file, use `path` + `glob: "<File>.cls"`.
- `list_conversations` and the other workspace tools take `workspaceId`, **not** `connectionId`. Filter by connection inside the arguments.
- `path` takes ONE folder; there is no exclude option. Custom metadata records (`*__mdt`) are not indexed; find them via the Apex that reads them.
- The `Report` folder may not be indexed. Check `get_index_status` before searching reports.

## DX0 coverage gaps
DX0 doesn't index CallCenter XML, SoftphoneLayout, Connected Apps, Reports/Dashboards or managed-package Apex internals, and doesn't return Custom Metadata record values or managed-package config records (query those on the record connector). Name the gap whenever it matters.

## More traps
- Several record connectors (production plus sandboxes) use the same tool names under different server prefixes. Always use the one whose `getUserInfo` username matches the DX0 connection being analyzed.
- Sandbox schema can differ from production (fields that exist only in one). Check schema in the org you query.
- Sandbox SOSL indexes can lag: if `find` returns nothing for a value you expect, retry with SOQL on a Name or external-Id filter.
- `OpportunityFieldHistory` only holds tracked fields; zero rows on a tracked field means the value was never set and cleared.
- Managed-package config (e-signature field mappings and templates, CPQ price rules) lives in records, not indexed metadata: query it.
- `__mdt` records and some setup objects may not be queryable through the record connector (no Tooling API). Test once; if blocked, draft the query.
- No Flow or setup-metadata access through the record connector, and it can't read ApexLog bodies: when only a debug log can settle the cause, ask the user for it.
- Text formula "booleans" return the strings "True"/"False"; compare as text.

## Gotchas from live runs
- **Validation rules:** check them on "can't save" or "can't change stage" symptoms. A live run missed one by looking only at Flows and Apex.
- **Writers of a field:** for "reverts" bugs, search Apex as well as Flows. A live ground truth was an Apex trigger path (a `before upsert` method forcing a stage), not the Flows an analyst first suspected.
- **Inactive triggers:** `signal_multipleTriggers` reports them as critical. Ignore `status: Inactive`.
- **Labels aren't API values:** UI labels, batch names and display combinations ("New/Won") often have zero metadata hits. Search for the API value or the code that sets it.
- **Integration logic:** when the change lives in middleware, an empty search is expected. Say so.
- **Managed packages:** namespaced objects hide their internals.
- **The snapshot is current state:** for closed tickets, the fix may already be in the indexed code.
- **Missed reuse is the costliest mistake:** in testing, the worst designs proposed a new batch or field when an existing flow, batch job, or metadata-driven updater already did the job. Run the reuse sweep before anything else.
- **Picklist mismatch:** report and Apex filters often hard-code values (e.g. `Type = 'New Business'`) that a record type never uses. On "missing from report" bugs, compare the hard-coded values with the values the records actually hold.
