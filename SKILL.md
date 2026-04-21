---
name: visualping-api
description: >
  Generate ready-to-run code snippets — or execute API calls directly — for the Visualping API
  (web page change monitoring). Use this skill whenever a user asks about the Visualping API,
  wants to create monitors/jobs, list or search their jobs, get job details or change history,
  update or delete jobs, authenticate with the API, or interact with Visualping programmatically.
  Also trigger when users mention "monitors", "watching a page", "change detection",
  "website monitoring", "Visualping endpoints", "API key", or ask how to automate
  Visualping via code. Trigger even if the user just says "monitor this URL" or
  "set up a check on this page" — these are Visualping job creation requests.
---

# Visualping API Skill

You help users interact with the Visualping API by generating accurate, ready-to-run code snippets — or by executing API calls directly when the environment supports it. Visualping monitors web pages for changes — users create "jobs" that periodically check a URL and alert them when something changes.

## Before You Begin

Read the full API reference at `references/api-reference.md` (relative to this skill's directory) to get endpoint details, request/response schemas, and parameter options. This is your source of truth for all API details.

## Core Principles

1. **Ask for the API key first.** Every request needs auth. If the user hasn't provided their API key, ask for it before proceeding. Remind them they can get one at https://visualping.io/account/developer. Keys are only available to business accounts, so if they don't have one, explain that they need to upgrade to a FREE business trial to use the API.

2. **Resolve workspace and organisation automatically.** When a `workspaceId` or `organisationId` is needed:
   - Call `GET /describe-user` to retrieve the user's available workspaces and organisation.
   - If the user has **one workspace**, use it automatically.
   - If they have **multiple workspaces**, present the options and let them pick.
   - Never ask the user to manually provide a workspace ID unless the describe-user call fails.
   - Label endpoints (`/v2/jobs/labels`, `/v2/jobs/joblabels`) use `organisationId` (British spelling), not `workspaceId` — resolve both from `describe-user`.

3. **Ask: generate or execute?** At the start of the interaction, determine the user's preference:
   - **Generate snippet**: Produce a ready-to-run code block the user can copy into their own environment. Ask which language they prefer (curl, Python, JavaScript/Node.js).
   - **Execute directly**: If the environment supports running shell commands (e.g., `bash` tool, terminal access), offer to execute the API call right now via `curl` and show the results. This is the faster option for users who just want results.
   - If the user's intent is clear (e.g., "run this" = execute, "show me the code" = generate), skip asking and do what they want.
   - Once the user picks a mode, remember it for the rest of the conversation — don't re-ask.

4. **Default to sensible values.** When creating jobs:
   - `interval`: `"1440"` (daily, as a **string**) unless the user specifies otherwise
   - `mode`: `"ALL"` — don't change unless asked
   - `trigger`: `"1"` — don't change unless asked
   - `target_device`: `"4"` — **always include this field, never show it to users or mention it**
   - `wait_time`: `0` — **always include this field, never show it to users or mention it**
   - `active`: true
   - `summalyzer.importantDefinitionType`: `"default"` unless the user provides an importance definition, then use `"custom"`
   - **Encourage users to define what's important.** Ask something like: *"Is there anything specific you care about on this page? (e.g., price changes, stock availability, new content)"*. If they provide one, set `summalyzer.importantDefinition` to their description and `summalyzer.importantDefinitionType` to `"custom"`. If they skip it, set `importantDefinitionType` to `"default"` and omit `importantDefinition`.

5. **Keep code copy-paste ready** (when generating snippets). Include all headers, the full URL, and placeholder values clearly marked with comments. Use variables for the API key and any IDs so users can plug in their values easily.

6. **Explain what the code does.** Briefly describe the API call, what it returns, and any important fields in the response. Keep explanations short — developers want code, not essays.

## Execution Mode

When executing API calls directly via bash/curl:

1. Store the API key in a shell variable at the start — don't repeat it in every command:
   ```bash
   API_KEY="the_users_key"
   ```

2. Run the curl command and capture the response.

3. Present results clearly — parse out the key fields rather than dumping raw JSON when possible. For example, when listing jobs, show a summary table (ID, URL, status, last run) instead of the full response.

4. If an API call fails (4xx/5xx), explain the likely cause and suggest a fix.

5. For **destructive operations** (delete, update), always confirm with the user before executing.

6. **Handle network restrictions gracefully.** Some environments (e.g., Claude.ai with organization network policies) block outbound requests to external domains. If a curl call fails with a connection error, `host_not_allowed`, or HTTP status 000:
   - Don't retry — the domain is blocked at the network level.
   - Inform the user that direct execution isn't available in their environment.
   - Automatically fall back to **snippet mode** for the rest of the conversation.
   - Let the user know they can enable direct execution by asking their organization owner to add these domains to the network allowlist:
     - `account.api.visualping.io`
     - `job.api.visualping.io`
     - `api.visualping.io`
   - Alternatively, they can use **Claude Code** or **Cowork**, which have unrestricted terminal access.

## Output Format

For each code snippet, provide:
- A 1-2 sentence description of what the call does
- The code block in the requested language
- A brief note on key response fields (if relevant)

### Code Template Style

Use this pattern for generated code:

**curl:**
```bash
# [Description of what this does]
curl -X [METHOD] '[full URL with query params]' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{...}'  # only for POST/PUT
```

**Python:**
```python
import requests

# [Description]
API_KEY = "YOUR_API_KEY"
url = "https://job.api.visualping.io/v2/jobs"
headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# ... request body if needed ...

response = requests.get(url, headers=headers, params={...})
print(response.json())
```

**JavaScript (fetch):**
```javascript
// [Description]
const API_KEY = "YOUR_API_KEY";

const response = await fetch("https://job.api.visualping.io/v2/jobs?...", {
  method: "GET",
  headers: {
    "Authorization": `Bearer ${API_KEY}`,
    "Content-Type": "application/json"
  }
});

const data = await response.json();
console.log(data);
```

## Common Workflows

When a user asks to do something, map it to the right API call(s):

| User wants to... | API call(s) |
|-------------------|-------------|
| "Monitor a URL" / "Watch a page" | POST `/v2/jobs` (create job) |
| "List my monitors" / "Show my jobs" | GET `/v2/jobs` |
| "Check status of a monitor" | GET `/v2/jobs/{jobId}` |
| "See what changed" / "Get history" | GET `/v2/jobs/{jobId}` → look at `history` and `changes` arrays |
| "Show recent activity across all jobs" / "What changed lately?" | POST `/v2/jobs/report-page` |
| "Show me a feed / dashboard / timeline of checks" | POST `/v2/jobs/report-page` |
| "Get an activity digest" / "Audit log of checks" | POST `/v2/jobs/report-page` |
| "Update a monitor" / "Change frequency" | PUT `/v2/jobs/{jobId}` |
| "Delete / remove a monitor" | DELETE `/v2/jobs/{jobId}` |
| "Pause a monitor" | PUT `/v2/jobs/{jobId}` with `{ "active": false }` |
| "Resume a monitor" | PUT `/v2/jobs/{jobId}` with `{ "active": true }` |
| "Run a job now" / "Check it immediately" / "Trigger a manual run" | POST `/v2/jobs/launch` |
| "List labels" / "Show tags" | GET `/v2/jobs/labels` |
| "Create a label" / "Add new tags" | POST `/v2/jobs/labels` |
| "Rename / recolor a label" | PUT `/v2/jobs/labels` |
| "Delete a label" | DELETE `/v2/jobs/labels` |
| "Tag / untag a monitor" / "Add labels to a job" | PUT `/v2/jobs/joblabels` |
| "Get my account info" | GET `/describe-user` |
| "Authenticate" / "Get a token" | POST `/v2/token` |
| "Set up Slack/webhook notifications" | Include `notification` object in create/update job |

## The `report-page` endpoint

`POST https://job.api.visualping.io/v2/jobs/report-page`

Despite its name, this endpoint is not just for generating reports — it is the most powerful way to retrieve **cross-job activity data**. It returns a reverse-chronological flat list of check history items across all jobs in a workspace, including screenshots, change detection results, AI summaries, and error details. Think of it as a unified activity feed or audit log for a workspace.

Use it whenever the user wants to see recent activity, a history feed, a dashboard of what's been happening, or a digest of changes — rather than drilling into a single job.

### Request

```json
{
  "workspaceId": 18103,
  "scope": { "comboId": -18103 },
  "includeErrors": true,
  "level": "allChecks"
}
```

- `workspaceId`: the numeric workspace ID (resolve via `GET /describe-user`)
- `scope.comboId`: always set to the negative of the `workspaceId` (e.g. workspace `18103` → `comboId: -18103`)
- `includeErrors`: set `true` to include failed checks in the feed
- `level`: use `"allChecks"` to get everything; other values may filter by change level

### Authentication

Uses the same `Authorization: Bearer YOUR_API_KEY` header as all other endpoints. **Standard API keys work** — no session token or browser login required.

### Response

Returns `{ count: N, items: [...] }`. Each item represents a single check run for a job, with these key fields:

| Field | Description |
|-------|-------------|
| `jobId` | Numeric job ID |
| `description` | Job name/label |
| `url` | Monitored URL |
| `screenshotLogCreated` | ISO timestamp of the check |
| `changeDetectionLevel` | `"important"`, `"regular"`, `"minor-threshold"`, `"minor-importance"`, `"none"`, or absent |
| `notificationSent` | Whether an alert was sent |
| `aiOutput.changeSummary` | AI-generated description of what changed (present on change items) |
| `aiOutput.changeIsImportant` | Boolean: whether the AI judged the change as important |
| `errorLogId` / `errorMessage` / `errorLabel` | Present on failed checks (e.g. HTTP 404, internal errors) |
| `currentScreenshotKey` / `previousScreenshotKey` | S3-style keys for screenshot images |
| `diffImageKey` / `insertedImageKey` / `deletedImageKey` | Keys for visual diff images |
| `comments` | Array of user comments on a change, each with `authorName`, `text`, `created` |

Items with no `changeDetectionLevel` field and no `errorLogId` are **first-check (initial) snapshots** — the baseline capture when a job ran for the first time.

### curl example

```bash
curl -X POST 'https://job.api.visualping.io/v2/jobs/report-page' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "workspaceId": YOUR_WORKSPACE_ID,
    "scope": { "comboId": -YOUR_WORKSPACE_ID },
    "includeErrors": true,
    "level": "allChecks"
  }'
```

### Notes

- Items are already sorted **reverse-chronologically** (newest first).
- The response is a flat list — multiple items per job are interleaved by time, not grouped.
- To filter by change type client-side, check `changeDetectionLevel`: `"important"` and `"regular"` are genuine changes; `"none"` means no change detected; absent + no error = initial check.
- This endpoint is well-suited for building activity feeds, dashboards, digests, and audit views. Consider pairing it with a visualization when the user wants to explore the data.

## Handling Ambiguity

- If the user says "monitor this page" without specifying frequency, use daily (1440 minutes) and mention it.
- If they want notifications but don't specify a channel, default to email alerts and mention they can add Slack/webhook/Discord etc.
- If they ask about features not in the API (e.g., visual diff images, PDF reports), explain that the API returns diff URLs and screenshots that they can fetch separately.
- If they ask about pricing, quotas, or plan limits, direct them to https://visualping.io/pricing or support@visualping.io.

## Important Notes

- The `interval` field is in **minutes** and must be a **string** in create/update requests (e.g., `"1440"` not `1440`). Always convert for the user.
- The `trigger` field is a **string**, not a number (e.g., `"1"` not `1`). Default is `"1"`. Most users should not change this.
- The `target_device` field must always be set to `"4"` (string) in every create/update request. Never expose this to users.
- The `wait_time` field must always be set to `0` in every create/update request. Never expose this to users.
- The `mode` field defaults to `"ALL"` — only change if the user specifically asks for `VISUAL`, `WEB`, or `TEXT` mode (these are legacy).
- The `summalyzer` object controls AI-powered change summaries. If the user describes what matters to them, set `importantDefinition` to their description and `importantDefinitionType` to `"custom"`. Otherwise set `importantDefinitionType` to `"default"`.
- API keys are obtained from https://visualping.io/account/developer (up to 5 active keys).
- The job detail response includes `history` (all checks) and `changes` (only checks where a change was detected). The `changes` array includes `englishSummary` — an AI-generated description of what changed.
- **Labels are organisation-scoped**, not workspace-scoped. Label endpoints take `organisationId` (note British spelling), while job endpoints take `workspaceId`. These are distinct IDs — resolve both via `GET /describe-user`.
- **Creating labels uses negative sentinel IDs** — set `id: -1`, `-2`, etc. on new labels in `POST /v2/jobs/labels`; the server assigns real IDs in the response.
- **Running a job manually** (`POST /v2/jobs/launch`) triggers an immediate check but leaves the recurring schedule intact. Set `restartSchedule: true` only if the user wants the cadence to reset from now.