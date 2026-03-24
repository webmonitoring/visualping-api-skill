# Visualping API Reference

## Base URLs

| Service        | Base URL                              |
|----------------|---------------------------------------|
| Authentication | `https://api.visualping.io`           |
| Jobs (CRUD)    | `https://job.api.visualping.io`       |
| Account/User   | `https://account.api.visualping.io`   |

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
  "id_token": "eyJ...",       // valid 24 hours
  "refresh_token": "eyJ..."   // valid 30 days
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

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| workspaceId | integer | Yes (business users) | — | Workspace ID |
| mode | string | No | `normal` | Output mode: `counts_only`, `ids_and_wsIds`, `ids_only`, `normal` |
| pageIndex | integer | No | 0 | Page number (0-based) |
| pageSize | integer | No | 100 | Jobs per page (min 1) |
| activeFilter | array[int] | No | — | Filter: `0` (paused), `1` (active) |
| modeFilter | array[string] | No | — | Filter by mode: `VISUAL`, `WEB`, `TEXT` |
| frequencyFilter | array[string] | No | — | Filter by frequency bucket |
| hasAdvancedScheduleFilter | int (0/1) | No | — | Filter by advanced schedule presence |
| eventFilter | string | No | — | Filter by event: `changed`, `changedImportant`, `errored`, `checked` |
| dateFilter | string | No | — | Time interval for eventFilter |
| dateFilterStart | string | No | — | ISO date, required if dateFilter is `since_custom_date` |
| fullTextSearchFilter | string | No | — | Search in URLs/descriptions |
| labelsFilter | array[int] | No | — | Filter by label IDs |
| sortBy | array[string] | No | `active_first,lastrun_desc` | Sort order |

**sortBy options:** `id_asc`, `id_desc`, `created_asc`, `created_desc`, `lastrun_asc`, `lastrun_desc`, `active_first`, `inactive_first`, `frequency_asc`, `frequency_desc`, `alphabetical_asc`, `alphabetical_desc`, `last_diff_detected_asc`, `last_diff_detected_desc`

---

### 3. Create Job
```
POST https://job.api.visualping.io/v2/jobs
Authorization: Bearer <token>
Content-Type: application/json
```

#### Request Body

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| workspaceId | integer | Yes (business) | — | Workspace ID |
| url | string | **Yes** | — | URL to monitor |
| description | string | No | — | Human-readable label for the job |
| mode | string | No | `ALL` | Monitor mode. Options: `VISUAL`, `WEB`, `TEXT`, `ALL`. Default `ALL` — most users should not change this. |
| active | boolean | No | true | Whether the job starts active |
| interval | string | No | `"1440"` | Check frequency in **minutes** (as a string). Default `"1440"` (once per day). Common values: `"5"`, `"15"`, `"30"`, `"60"`, `"360"`, `"720"`, `"1440"`. |
| trigger | string | No | `"1"` | Sensitivity threshold. Default `"1"`. Most users should not change this. |
| crop | object | No | — | Screen region to monitor: `{ x, y, width, height }` |
| proxy_id | integer | No | — | Proxy to use |
| xpath | string | No | — | XPath or CSS selector to target a specific element |
| keyword_action | string | No | `ALL` | Keyword filter mode |
| keywords | string | No | — | Keywords to filter changes |
| disable_js | boolean | No | false | Disable JavaScript rendering |
| enable_cookies_and_ad_blocker | boolean | No | false | Enable cookies and ad blocking |
| wait_time | integer | No | 0 | Seconds to wait after page load before screenshot |
| preactions | object | No | — | Actions to perform before checking (click, type, etc.) |
| advanced_schedule | object | No | — | Restrict checks to specific times/days |
| notification | object | No | — | Notification configuration (see below) |
| retention_policy | string | No | `"3"` | History retention policy |
| alert_error | boolean | No | true | Alert on errors |
| summalyzer | object | No | — | AI summary configuration |
| labelIds | array[int] | No | — | Labels to attach |

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

```json
{
  "importantDefinition": "Describe what changes matter to you",
  "importantDefinitionType": "custom"
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

### 4. Get Job Details & History
```
GET https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
```

#### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| jobId | integer | Yes | Job ID |

#### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
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

### 5. Update Job
```
PUT https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
Content-Type: application/json
```

Request body is the same shape as Create Job. Only include fields you want to change.

---

### 6. Delete Job
```
DELETE https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
```

#### Query Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| workspaceId | integer | Business users | Workspace ID |

---

## Common Interval Values

| Interval (string) | Human-readable |
|---------------------|----------------|
| `"5"` | Every 5 minutes |
| `"15"` | Every 15 minutes |
| `"30"` | Every 30 minutes |
| `"60"` | Every hour |
| `"360"` | Every 6 hours |
| `"720"` | Every 12 hours |
| `"1440"` | Once a day (default) |
| `"10080"` | Once a week |

## Hidden Defaults

These fields must always be included in create/update requests but should never be shown to or configured by users:

| Field | Value | Notes |
|-------|-------|-------|
| `target_device` | `"4"` | Always include as string `"4"`. Do not expose to users. |

## HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request / invalid parameters |
| 403 | Forbidden / auth failure |
| 500 | Server error |
