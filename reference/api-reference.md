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

Returns information about the authenticated user, including their profile and workspaces. Business users also receive an `organisation` section.

#### Response Fields

| Field                     | Type    | Description                                                |
| ------------------------- | ------- | ---------------------------------------------------------- |
| `userId`                  | integer | Unique user ID                                             |
| `emailAddress`            | string  | User's email address                                       |
| `country`                 | string  | User's country                                             |
| `locale`                  | string  | Locale code (e.g. `en`)                                    |
| `timeZone`                | string  | User's time zone (e.g. `America/Vancouver`)                |
| `loginCount`              | integer | Total number of logins                                     |
| `signUpAgeDays`           | integer | Days since sign-up                                         |
| `intentForBusiness`       | boolean | Whether the user signed up for business use                |
| `googleChatIntegrated`    | boolean | Google Chat integration status                             |
| `googleSheetsIntegrated`  | boolean | Google Sheets integration status                           |
| `slackInstallationStatus` | object  | Slack installation details                                 |
| `refreshToken`            | boolean | Whether a refresh token is present                         |
| `organisation`            | object  | Organisation details — **business users only** (see below) |
| `workspaces`              | array   | List of workspaces the user belongs to (see below)         |

##### `organisation` object

| Field                         | Type              | Description                                    |
| ----------------------------- | ----------------- | ---------------------------------------------- |
| `id`                          | integer           | Organisation ID                                |
| `name`                        | string            | Organisation name                              |
| `timeZone`                    | string            | Organisation time zone                         |
| `role`                        | string            | User's role in the organisation (e.g. `ADMIN`) |
| `plan`                        | object            | Current subscription plan details              |
| `userHasPersonalSubscription` | boolean           | Whether the user has a personal subscription   |
| `accountFeatures`             | object            | Feature flags and limits for the organisation  |
| `balances`                    | object            | Credit balance and renewal info                |
| `counts`                      | object            | Active job and user counts                     |
| `trialStartDate`              | string (ISO 8601) | Trial start date                               |
| `trialEndDate`                | string (ISO 8601) | Trial end date                                 |
| `orgCreatedAt`                | string (ISO 8601) | Organisation creation date                     |

##### `workspaces` array items

| Field                 | Type    | Description                                       |
| --------------------- | ------- | ------------------------------------------------- |
| `id`                  | integer | Workspace ID                                      |
| `name`                | string  | Workspace name                                    |
| `timeZone`            | string  | Workspace time zone                               |
| `role`                | string  | User's role in this workspace (e.g. `ADMIN`)      |
| `plan`                | object  | Workspace subscription plan                       |
| `accountFeatures`     | object  | Feature flags and limits for this workspace       |
| `balances`            | object  | Credit balance including `inOverConsumption` flag |
| `counts`              | object  | Active job and user counts                        |
| `notificationMembers` | array   | List of CC email recipients for notifications     |
| `status`              | string  | Workspace status (e.g. `active`)                  |

#### Example Response

```json
{
  "userId": 123456,
  "emailAddress": "user@example.com",
  "country": "Canada",
  "locale": "en",
  "timeZone": "America/Vancouver",
  "loginCount": 77,
  "slackInstallationStatus": {},
  "refreshToken": false,
  "organisation": {
    "id": 137388,
    "name": "Visualping",
    "timeZone": "America/Vancouver",
    "role": "ADMIN",
    "plan": {
      "name": "trial_2022.1_active",
      "type": "plan",
      "subType": "trial_active",
      "credits": 500,
      "billingPeriod": "monthly",
      "label": "Free trial"
    },
    "userHasPersonalSubscription": false,
    "accountFeatures": {
      "reports": { "enabled": true },
      "exports": { "enabled": true },
      "bulkImport": { "enabled": true },
      "ccEmails": { "enabled": true, "maxAddressCount": 5 },
      "sms": { "enabled": true },
      "advancedNotifications": { "enabled": true },
      "maxActiveJobsPerWorkspace": { "value": 25 },
      "maxJobFrequency": { "value": 1 },
      "emailSupport": { "enabled": true },
      "dedicatedSupport": { "enabled": true }
    },
    "balances": {
      "nextCreditRenewalAt": "2026-06-11T19:19:59.170Z",
      "estimatedMonthlyConsumption": 774
    },
    "counts": {
      "activeJobCount": 2,
      "activeJobFreeCount": 23,
      "activeJobOverflow": false,
      "activeUserCount": 4,
      "activeUserFreeCount": 6,
      "activeUserOverflow": false
    },
    "trialStartDate": "2026-05-07T19:19:59.170Z",
    "trialEndDate": "2026-05-21T19:19:59.170Z",
    "orgCreatedAt": "2026-05-07T19:19:59.000Z"
  },
  "workspaces": [
    {
      "id": 137389,
      "name": "Workspace #1",
      "timeZone": "America/Vancouver",
      "role": "ADMIN",
      "plan": { "name": "trial_2022.1_active", "label": "Free trial" },
      "balances": {
        "credits": 146,
        "nextCreditRenewalAt": "2055-11-21T19:19:59.170Z",
        "estimatedMonthlyConsumption": 732,
        "inOverConsumption": true
      },
      "counts": {
        "activeJobCount": 1,
        "activeJobFreeCount": 499999,
        "activeJobOverflow": false
      },
      "notificationMembers": [{ "type": "ccEmail", "value": "user@example.com", "confirmed": true }],
      "status": "active"
    }
  ]
}
```

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

| Field | Type | Required | Default  | Description                                                                                                                                              |
|-------|------|----------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| workspaceId | integer | Yes (business) | —        | Workspace ID                                                                                                                                             |
| url | string | **Yes** | —        | URL to monitor                                                                                                                                           |
| description | string | No | —        | Human-readable label for the job                                                                                                                         |
| mode | string | No | `ALL`    | Monitor mode. Options: `VISUAL`, `WEB`, `TEXT`, `ALL`. Default `ALL` — most users should not change this.                                                |
| active | boolean | No | true     | Whether the job starts active                                                                                                                            |
| interval | string | No | `"1440"` | Check frequency in **minutes** (as a string). Default `"1440"` (once per day). Common values: `"5"`, `"15"`, `"30"`, `"60"`, `"360"`, `"720"`, `"1440"`. |
| trigger | string | No | `"1"` | Sensitivity threshold. Default `"1"`. Most users should not change this. |
| crop | object | No | — | Screen region to monitor: `{ x, y, width, height }` |
| proxy_id | integer | No | — | Proxy to use |
| xpath | string | No | — | XPath or CSS selector to target a specific element |
| keyword_action | string | No | `ALL` | Keyword filter mode |
| keywords | string | No | — | Keywords to filter changes |
| disable_js | boolean | No | false | Disable JavaScript rendering |
| enable_cookies_and_ad_blocker | boolean | No | false | Enable cookies and ad blocking |
| preactions | object | No | — | Actions to perform before checking (click, type, etc.) |
| advanced_schedule | object | No | — | Restrict checks to specific times/days |
| notification | object | No | — | Notification configuration (see below) |
| retention_policy | string | No | `"3"` | History retention policy |
| alert_on_error | object\|null | No | default threshold | Error alerting configuration (see below) |
| alert_error | boolean | No | — | **Deprecated** alias for `alert_on_error`. `true` → `{ "change_baseline": true }`, `false` → reset to the default threshold (it cannot turn error alerts off). Ignored when `alert_on_error` is present in the same request. |
| summalyzer | object | No | — | AI summary configuration (see below) |
| labelIds | array[int] | No | — | Labels to attach. See §11–15 for managing labels and bulk job↔label assignment. |

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

#### Alert on Error Object

`alert_on_error` controls whether/how the user is notified when checks fail. It is a **mutually exclusive union** — send exactly one of these shapes:

| Value | Behavior |
|-------|----------|
| `{ "threshold": N }` | Send an error notification each time the consecutive-error streak reaches a multiple of `N` (N, 2N, 3N, ...). The baseline is untouched and the streak resets when the job succeeds again. `N` is an integer from 1 to 1000 (`1` = alert on every errored check). |
| `{ "never": true }` | Never send error notifications. |
| `{ "change_baseline": true }` | Legacy behavior: the error page is crawled as page content, the baseline moves, and the user gets an ordinary change-detection alert. |
| `null` | Reset to the default (see below). |

Combining fields (e.g. `threshold` + `never`) is rejected with a 400.

**Default:** when `alert_on_error` was never configured, the job follows a **server-side default threshold** (currently 100 consecutive errors) — absence does **not** mean "never". The only way to disable error alerts is an explicit `{ "never": true }`; the deprecated `"alert_error": false` does **not** disable them, it resets the job to the default threshold. Setting a threshold equal to the current default is stored as "follow the default", so such a job tracks the default if it later changes and `GET` returns no `alert_on_error` for it.

**Delivery:** error alerts go to the job's configured notification channels (email, SMS, webhook, Slack, Teams, Discord, Telegram, Google Chat) — Google Sheets is excluded. The `onlyImportantAlerts` flag does not apply to error alerts. Webhook error payloads carry the standard change-payload fields plus `event: "error"`, `error_label`, `error_message`, and `consecutive_errors` (change payloads carry `event: "change"`); diff/screenshot fields are present when the errored check produced a diff against the baseline.

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

### 5. Bulk Create Jobs

```
PUT https://job.api.visualping.io/v2/jobs/bulk
Authorization: Bearer <token>
Content-Type: application/json
```

Creates many jobs in a single request, applying a shared `settings` block to every URL in the `jobs` array. Use this for importing lists of URLs (e.g. CSV upload, onboarding) where every job should share the same monitoring configuration.

Despite using `PUT`, this endpoint **creates** new jobs — it does not update existing ones.

#### Request Body

| Field        | Type          | Required       | Description                                                                                       |
| ------------ | ------------- | -------------- | ------------------------------------------------------------------------------------------------- |
| settings     | object        | Yes            | Shared configuration applied to every job. Same shape as Create Job body (without `url`).         |
| jobs         | array[object] | Yes            | One entry per job to create. Each entry must include `url`; per-job overrides are also supported. |
| workspaceId  | integer       | Yes (business) | Workspace where the jobs will be created                                                          |
| initToBlank  | boolean       | No             | If `true`, jobs start with a blank baseline (no initial screenshot used as reference)             |

The `settings` object accepts every field documented under §3 Create Job (e.g. `mode`, `interval`, `trigger`, `notification`, `preactions`, `labelIds`, `enable_cookies_and_ad_blocker`, `target_device`, `wait_time`, etc.).

Each entry in `jobs` must contain at least `url`. Any field accepted by Create Job may be included on a per-job basis to override the shared `settings` for that one URL (e.g. a custom `description` or `xpath`).

#### Request Example

```json
{
  "workspaceId": 4139,
  "initToBlank": false,
  "settings": {
    "active": true,
    "mode": "ALL",
    "interval": "1440",
    "trigger": "0.01",
    "notification_threshold": 0.01,
    "target_device": "4",
    "wait_time": 0,
    "enable_cookies_and_ad_blocker": true,
    "alert_on_error": { "never": true },
    "labelIds": [142819],
    "preactions": {
      "active": true,
      "actions": [{ "click": "home" }]
    },
    "notification": {
      "enableSmsAlert": false,
      "enableEmailAlert": true,
      "onlyImportantAlerts": true,
      "config": {
        "webhook": { "url": "", "active": false, "notificationType": "webhook" }
      }
    },
    "summalyzer": { "importantDefinitionType": "none" }
  },
  "jobs": [
    { "url": "https://example.com/page-1" },
    { "url": "https://example.com/page-2", "description": "Pricing page" }
  ]
}
```

---

### 6. Get Job Details & History

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
  "retention_policy": "3",
  "alert_on_error": { "threshold": 5 },
  "alert_error": false
}
```

`alert_on_error` is the configured error-alerting setting (see §3). It is **absent when the job was never configured**, meaning the job follows the server-side default threshold. `alert_error` is the deprecated boolean alias: `true` only when the setting is `{ "change_baseline": true }`.

---

### 7. Update Job

```
PUT https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
Content-Type: application/json
```

Request body is the same shape as Create Job. Only include fields you want to change.

For `alert_on_error`: omitting the field leaves the current setting unchanged; sending `null` resets the job to the default error-alert threshold; sending `{ "never": true }` turns error alerts off.

---

### 8. Delete Job

```
DELETE https://job.api.visualping.io/v2/jobs/{jobId}
Authorization: Bearer <token>
```

#### Query Parameters

| Parameter   | Type    | Required       | Description  |
| ----------- | ------- | -------------- | ------------ |
| workspaceId | integer | Business users | Workspace ID |

---

### 9. Report Page (Activity Feed)

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

### 10. Launch Job (Run Now)
```
POST https://job.api.visualping.io/v2/jobs/launch
Authorization: Bearer <token>
Content-Type: application/json
```

Manually triggers an immediate check for one or more jobs.

#### Request Body
| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| jobIds | array[int] | Yes | — | Job IDs to run now |
| workspaceId | integer | Business | — | Workspace ID |
| restartSchedule | boolean | No | false | If true, the job's recurring schedule is reset to start from now |

#### Example
```json
{ "jobIds": [6024347], "restartSchedule": false, "workspaceId": 2456 }
```

---

### 11. List Labels
```
GET https://job.api.visualping.io/v2/jobs/labels
Authorization: Bearer <token>
```

Labels are scoped to the **organisation**, not the workspace.

#### Query Parameters
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| organisationId | integer | Yes | — | Organisation ID (note British spelling) |
| sortBy | string | No | — | e.g. `alphabetical_asc`, `alphabetical_desc` |
| computeUsage | int (0/1) | No | 0 | If `1`, each label includes `numOfJobsLabeled` |

---

### 12. Create Labels
```
POST https://job.api.visualping.io/v2/jobs/labels
Authorization: Bearer <token>
Content-Type: application/json
```

Creates one or more labels in a single request.

#### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| organisationId | integer | Yes | Organisation ID |
| labels | array[object] | Yes | Labels to create |

**Label object (for creation):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | integer | Yes | Use a **negative sentinel** (`-1`, `-2`, …) to indicate a new label; the server assigns the real ID |
| name | string | Yes | Label name |
| color | string | Yes | Hex color, e.g. `"#FFFAEB"` |
| emoji | string | No | Emoji shortcode, e.g. `":new:"` |

#### Example
```json
{
  "organisationId": 2455,
  "labels": [
    { "id": -1, "name": "tmp",  "color": "#FFFAEB", "emoji": ":new:" },
    { "id": -2, "name": "tmp2", "color": "#FDF2FA", "emoji": ":new:" }
  ]
}
```

---

### 13. Update Labels
```
PUT https://job.api.visualping.io/v2/jobs/labels
Authorization: Bearer <token>
Content-Type: application/json
```

#### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| organisationId | integer | Yes | Organisation ID |
| labels | array[object] | Yes | Labels to update; each must use the real (positive) `id` |

`numOfJobsLabeled` is accepted in the payload but is effectively read-only.

#### Example
```json
{
  "organisationId": 2455,
  "labels": [
    { "id": 9, "name": "tmp3", "emoji": ":new:", "color": "#FFFAEB", "numOfJobsLabeled": 0 }
  ]
}
```

---

### 14. Delete Label
```
DELETE https://job.api.visualping.io/v2/jobs/labels
Authorization: Bearer <token>
Content-Type: application/json
```

#### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | integer | Yes | Label ID to delete (single, not an array) |
| organisationId | integer | Yes | Organisation ID |

#### Example
```json
{ "id": 10, "organisationId": 2455 }
```

---

### 15. Assign / Unassign Labels on Jobs
```
PUT https://job.api.visualping.io/v2/jobs/joblabels
Authorization: Bearer <token>
Content-Type: application/json
```

Bulk add or remove labels across one or more jobs in a single call.

#### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| organisationId | integer | Yes | Organisation ID |
| jobIds | array[int] | Yes | Jobs to update |
| addLabelIds | array[int] | No | Label IDs to add |
| removeLabelIds | array[int] | No | Label IDs to remove |

#### Example
```json
{
  "organisationId": 2455,
  "jobIds": [6024347],
  "addLabelIds": [9, 10],
  "removeLabelIds": [12]
}
```

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

## HTTP Status Codes

| Code | Meaning                          |
| ---- | -------------------------------- |
| 200  | Success                          |
| 400  | Bad request / invalid parameters |
| 403  | Forbidden / auth failure         |
| 500  | Server error                     |
