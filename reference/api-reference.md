# Visualping API Reference

## Base URLs

| Service        | Base URL                            |
| -------------- | ----------------------------------- |
| Authentication | `https://api.visualping.io`         |
| Jobs (CRUD)    | `https://job.api.visualping.io`     |
| Account/User   | `https://account.api.visualping.io` |

## Authentication

All requests require a Bearer token in the `Authorization` header. Two auth methods:

### API Key (recommended)

Obtained from https://visualping.io/account/developer (up to 5 active keys).

```
Authorization: Bearer <apiKey>
```

### ID Token (email/password)

```
POST https://api.visualping.io/v2/token
Content-Type: application/json

{
  "method": "PASSWORD",
  "email": "user@company.com",
  "password": "your_password"
}
```

Response:

```json
{
  "id_token": "eyJ...", // valid 24 hours
  "refresh_token": "eyJ..." // valid 30 days
}
```

### Refresh Token

```
POST https://api.visualping.io/v2/token
Content-Type: application/json

{
  "method": "REFRESH_TOKEN",
  "refreshToken": "eyJ..."
}
```

Response:

```json
{
  "id_token": "eyJ..."
}
```

---

## Endpoints

### 1. Describe User

```
GET https://account.api.visualping.io/describe-user
Authorization: Bearer <token>
```

Returns information about the authenticated user.

---

### 2. List Jobs

```
GET https://job.api.visualping.io/v2/jobs
Authorization: Bearer <token>
```

#### Query Parameters

| Parameter                 | Type          | Required             | Default                     | Description                                                          |
| ------------------------- | ------------- | -------------------- | --------------------------- | -------------------------------------------------------------------- |
| workspaceId               | integer       | Yes (business users) | —                           | Workspace ID                                                         |
| mode                      | string        | No                   | `normal`                    | Output mode: `counts_only`, `ids_and_wsIds`, `ids_only`, `normal`    |
| pageIndex                 | integer       | No                   | 0                           | Page number (0-based)                                                |
| pageSize                  | integer       | No                   | 100                         | Jobs per page (min 1)                                                |
| activeFilter              | array[int]    | No                   | —                           | Filter: `0` (paused), `1` (active)                                   |
| modeFilter                | array[string] | No                   | —                           | Filter by mode: `VISUAL`, `WEB`, `TEXT`                              |
| frequencyFilter           | array[string] | No                   | —                           | Filter by frequency bucket                                           |
| hasAdvancedScheduleFilter | int (0/1)     | No                   | —                           | Filter by advanced schedule presence                                 |
| eventFilter               | string        | No                   | —                           | Filter by event: `changed`, `changedImportant`, `errored`, `checked` |
| dateFilter                | string        | No                   | —                           | Time interval for eventFilter                                        |
| dateFilterStart           | string        | No                   | —                           | ISO date, required if dateFilter is `since_custom_date`              |
| fullTextSearchFilter      | string        | No                   | —                           | Search in URLs/descriptions                                          |
| labelsFilter              | array[int]    | No                   | —                           | Filter by label IDs                                                  |
| sortBy                    | array[string] | No                   | `active_first,lastrun_desc` | Sort order                                                           |

**sortBy options:** `id_asc`, `id_desc`, `created_asc`, `created_desc`, `lastrun_asc`, `lastrun_desc`, `active_first`, `inactive_first`, `frequency_asc`, `frequency_desc`, `alphabetical_asc`, `alphabetical_desc`, `last_diff_detected_asc`, `last_diff_detected_desc`

---

### 3. Create Job

```
POST https://job.api.visualping.io/v2/jobs
Authorization: Bearer <token>
Content-Type: application/json
```

#### Request Body

| Field                         | Type       | Required       | Default  | Description                                                                                                                                              |
| ----------------------------- | ---------- | -------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| workspaceId                   | integer    | Yes (business) | —        | Workspace ID                                                                                                                                             |
| url                           | string     | **Yes**        | —        | URL to monitor                                                                                                                                           |
| description                   | string     | No             | —        | Human-readable label for the job                                                                                                                         |
| mode                          | string     | No             | `ALL`    | Monitor mode. Options: `VISUAL`, `WEB`, `TEXT`, `ALL`. Default `ALL` — most users should not change this.                                                |
| active                        | boolean    | No             | true     | Whether the job starts active                                                                                                                            |
| interval                      | string     | No             | `"1440"` | Check frequency in **minutes** (as a string). Default `"1440"` (once per day). Common values: `"5"`, `"15"`, `"30"`, `"60"`, `"360"`, `"720"`, `"1440"`. |
| trigger                       | string     | No             | `"1"`    | Sensitivity threshold. Default `"1"`. Most users should not change this.                                                                                 |
| crop                          | object     | No             | —        | Screen region to monitor: `{ x, y, width, height }`                                                                                                      |
| proxy_id                      | integer    | No             | —        | Proxy to use                                                                                                                                             |
| xpath                         | string     | No             | —        | XPath or CSS selector to target a specific element                                                                                                       |
| keyword_action                | string     | No             | `ALL`    | Keyword filter mode                                                                                                                                      |
| keywords                      | string     | No             | —        | Keywords to filter changes                                                                                                                               |
| disable_js                    | boolean    | No             | false    | Disable JavaScript rendering                                                                                                                             |
| enable_cookies_and_ad_blocker | boolean    | No             | false    | Enable cookies and ad blocking                                                                                                                           |
| preactions                    | object     | No             | —        | Actions to perform before checking (click, type, etc.)                                                                                                   |
| advanced_schedule             | object     | No             | —        | Restrict checks to specific times/days                                                                                                                   |
| notification                  | object     | No             | —        | Notification configuration (see below)                                                                                                                   |
| retention_policy              | string     | No             | `"3"`    | History retention policy                                                                                                                                 |
| alert_error                   | boolean    | No             | true     | Alert on errors                                                                                                                                          |
| summalyzer                    | object     | No             | —        | AI summary configuration (see below)                                                                                                                     |
| labelIds                      | array[int] | No             | —        | Labels to attach                                                                                                                                         |

#### Notification Object

```json
{
  "enableSmsAlert": true,
  "enableEmailAlert": true,
  "onlyImportantAlerts": true,
  "config": {
    "slack": { "url": "webhook_url", "active": true, "notificationType": "slack", "channels": ["#channel"] },
    "teams": { "url": "webhook_url", "active": true, "notificationType": "slack", "channels": ["#channel"] },
    "webhook": { "url": "https://your-endpoint.com", "active": true, "notificationType": "slack", "channels": [] },
    "discord": { "url": "webhook_url", "active": true, "notificationType": "slack", "channels": ["#channel"] },
    "slack_app": { "url": "...", "active": true, "notificationType": "slack", "channels": ["#channel"] },
    "google_sheets": { "url": "...", "active": true, "notificationType": "slack", "channels": [] },
    "google_chat": { "url": "...", "active": true, "notificationType": "slack", "channels": [] }
  }
}
```

#### Summalyzer (AI Summary) Object

| Field                     | Type   | Values                            | Description                                                                                           |
| ------------------------- | ------ | --------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `importantDefinitionType` | string | `"default"`, `"custom"`, `"none"` | Set to `"custom"` when user provides an importance definition. Otherwise `"default"`.                 |
| `importantDefinition`     | string | —                                 | User's description of what changes matter. Only include when `importantDefinitionType` is `"custom"`. |

**With user-defined importance (recommended — encourage users to provide this):**

```json
{
  "importantDefinition": "Price drops below $50 or item goes out of stock",
  "importantDefinitionType": "custom"
}
```

**Without (fallback):**

```json
{
  "importantDefinitionType": "default"
}
```

#### Advanced Schedule Object

```json
{
  "stop_time": 18,
  "start_time": 9,
  "active_days": [1, 2, 3, 4, 5]
}
```

Days: 1=Monday through 7=Sunday. Times are 0-24 (hours).

#### Preactions Object

```json
{
  "active": true,
  "actions": [
    { "action": "click", "selector": "#button-id" },
    { "action": "type", "selector": "#input-id", "value": "text to type" }
  ]
}
```

---

### 4. Create Job From Saved Settings

```
POST https://job.api.visualping.io/v2/jobs/from-saved-settings
Authorization: Bearer <token>
Content-Type: application/json
```

Creates a job by reusing saved job settings (preset/default preset).

#### Availability

- Business users only.
- Requires saved settings to exist for the organization that the workspace belongs to.
- If `savedJobSettingsId` is omitted, the workspace default preset is used.

#### Request Body

| Field              | Type    | Required       | Description                                                                                    |
| ------------------ | ------- | -------------- | ---------------------------------------------------------------------------------------------- |
| workspaceId        | integer | Yes (business) | ID of the workspace where the job will be added                                                |
| url                | string  | **Yes**        | URL to monitor                                                                                 |
| savedJobSettingsId | integer | No             | Saved settings ID to apply; if omitted, uses workspace default preset                          |
| description        | string  | No             | Human-readable label for the job                                                               |
| interval           | string  | No             | Optional override (minutes as string). Only include when explicitly overriding saved settings. |

#### Request Example

```json
{
  "workspaceId": 137389,
  "url": "https://example.com",
  "savedJobSettingsId": 12345,
  "description": "Weekly check from preset"
}
```

---

### 5. Get Job Details & History

```
GET https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
```

#### Path Parameters

| Parameter | Type    | Required | Description |
| --------- | ------- | -------- | ----------- |
| jobId     | integer | Yes      | Job ID      |

#### Query Parameters

| Parameter   | Type    | Required       | Description  |
| ----------- | ------- | -------------- | ------------ |
| workspaceId | integer | Business users | Workspace ID |

#### Response Body (key fields)

```json
{
  "id": "string",
  "url": "string",
  "description": "string",
  "mode": "VISUAL",
  "active": true,
  "interval": 1440,
  "runs": 0,
  "error_count": 0,
  "in_progress": true,
  "scheduled_at": "ISO date",
  "last_run": "ISO date",
  "next_run": "ISO date",
  "crop": { "x": 0, "y": 0, "width": 0, "height": 0 },
  "xpath": "string",
  "notification": { "..." },
  "advanced_schedule": { "..." },
  "thumb_full": "screenshot URL",
  "thumb_150": "thumbnail URL",
  "favicon": "favicon URL",
  "history": [
    {
      "created": "ISO date",
      "mode": "VISUAL",
      "PercentDifference": 0,
      "diff": {
        "nodePercentDiff": 0,
        "areaPercentDiff": 0,
        "wordPercentDiff": 0,
        "pixelPercentDiff": 0,
        "VISUAL": 0,
        "TEXT": 0
      },
      "image": "screenshot URL",
      "error_message": "string or null",
      "notification_send": true,
      "initial": false
    }
  ],
  "changes": [
    {
      "created": "ISO date",
      "mode": "VISUAL",
      "PercentDifference": 0,
      "diff": { "..." },
      "thumb_diff_full": "diff image URL",
      "htmlDiffUrl": "HTML diff URL",
      "englishSummary": "AI-generated summary of the change",
      "analyzerAlertTriggered": true,
      "feedback": { "DIFF": "GOOD", "SUMMARY": "GOOD" }
    }
  ],
  "summalyzer": { "..." },
  "labelIds": [0],
  "retention_policy": "3"
}
```

---

### 6. Update Job

```
PUT https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
Content-Type: application/json
```

Request body is the same shape as Create Job. Only include fields you want to change.

---

### 7. Delete Job

```
DELETE https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
```

#### Query Parameters

| Parameter   | Type    | Required       | Description  |
| ----------- | ------- | -------------- | ------------ |
| workspaceId | integer | Business users | Workspace ID |

---

### 8. Report Page (Activity Feed)

```
POST https://job.api.visualping.io/v2/jobs/report-page
Authorization: Bearer <token>
Content-Type: application/json
```

Returns a reverse-chronological flat list of check history items across all jobs in a workspace. Use this for activity feeds, dashboards, digests, and audit views.

#### Request Body

| Field         | Type    | Required | Description                                                              |
| ------------- | ------- | -------- | ------------------------------------------------------------------------ |
| workspaceId   | integer | Yes      | Workspace ID                                                             |
| scope.comboId | integer | Yes      | Negative of `workspaceId` (e.g. workspace `4348` → `-4348`)              |
| includeErrors | boolean | No       | Include failed checks in the feed (default: `false`)                     |
| level         | string  | No       | `"allChecks"` to get everything; other values may filter by change level |

#### Request Example

```json
{
  "workspaceId": 4348,
  "scope": { "comboId": -4348 },
  "includeErrors": true,
  "level": "allChecks"
}
```

#### Response Body

```json
{
  "count": 100,
  "items": [...]
}
```

Each item in `items` represents a single check run for a job:

| Field                              | Description                                                                                                |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `jobId`                            | Numeric job ID                                                                                             |
| `description`                      | Job name/label                                                                                             |
| `url`                              | Monitored URL                                                                                              |
| `screenshotLogId`                  | Unique ID for this check run                                                                               |
| `screenshotLogCreated`             | ISO timestamp of the check                                                                                 |
| `processId`                        | Internal processing ID                                                                                     |
| `changeDetectionLevel`             | `"important"`, `"regular"`, `"minor-threshold"`, `"minor-importance"`, `"none"`, or absent (initial check) |
| `notificationSent`                 | Whether an alert was triggered                                                                             |
| `aiOutput.changeSummary`           | AI-generated description of what changed                                                                   |
| `aiOutput.changeIsImportant`       | Boolean: whether the AI judged the change as important                                                     |
| `aiOutput.changeIsImportantReason` | Explanation of why the change was or wasn't flagged as important                                           |
| `matchedKeywords`                  | Array of any keywords that matched                                                                         |
| `currentScreenshotKey`             | S3-style key for the current screenshot                                                                    |
| `previousScreenshotKey`            | S3-style key for the previous screenshot                                                                   |
| `currentUncroppedScreenshotKey`    | S3-style key for the full-page current screenshot                                                          |
| `previousUncroppedScreenshotKey`   | S3-style key for the full-page previous screenshot                                                         |
| `diffImageKey`                     | Key for the visual diff image                                                                              |
| `insertedImageKey`                 | Key for inserted content image                                                                             |
| `deletedImageKey`                  | Key for deleted content image                                                                              |
| `faviconKey`                       | Key for the site favicon                                                                                   |
| `disableId`                        | Token used to unsubscribe from alerts                                                                      |
| `errorLogId`                       | Present on failed checks                                                                                   |
| `errorMessage`                     | Error description (if errored)                                                                             |
| `errorLabel`                       | Short error label (if errored)                                                                             |
| `comments`                         | Array of user comments: `{ authorName, text, created }`                                                    |

**Notes:**

- Items with no `changeDetectionLevel` and no `errorLogId` are initial baseline snapshots.
- Results are sorted newest-first and are interleaved across jobs (not grouped by job).
- The response is paginated at 100 items — additional pagination parameters may apply.

---

## Common Interval Values

| Interval (string) | Human-readable       |
| ----------------- | -------------------- |
| `"5"`             | Every 5 minutes      |
| `"15"`            | Every 15 minutes     |
| `"30"`            | Every 30 minutes     |
| `"60"`            | Every hour           |
| `"360"`           | Every 6 hours        |
| `"720"`           | Every 12 hours       |
| `"1440"`          | Once a day (default) |
| `"10080"`         | Once a week          |

## Hidden Defaults

These fields must always be included in create/update requests but should never be shown to or configured by users:

| Field           | Value | Notes                                                   |
| --------------- | ----- | ------------------------------------------------------- |
| `target_device` | `"4"` | Always include as string `"4"`. Do not expose to users. |
| `wait_time`     | `0`   | Always include as `0`. Do not expose to users.          |

## HTTP Status Codes

| Code | Meaning                          |
| ---- | -------------------------------- |
| 200  | Success                          |
| 400  | Bad request / invalid parameters |
| 403  | Forbidden / auth failure         |
| 500  | Server error                     |
