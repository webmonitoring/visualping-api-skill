# Visualping API

**Version: 1.2.0**

Documentation for API plus a skill that lets you interact with the [Visualping API](https://visualping.io) directly from LLMs — create monitors, list jobs, check change history, and more.

## What the skill does

- **Generate code snippets** (curl, Python, JavaScript) for any Visualping API call
- **Execute API calls directly** when running in an environment with terminal access
- Automatically resolves your workspace ID
- Uses sensible defaults so you can create monitors with minimal configuration

## Installation

### Option 1: Download the `.skill` file

1. Download `visualping-api.skill` from the [Releases](../../releases) page
2. In Claude, go to **Settings → Skills** (similar process should be available for other desktop LLM apps)
3. Upload the `.skill` file

### Option 2: Clone this repo

1. Clone the repository:
   ```bash
   git clone https://github.com/webmonitoring/visualping-api-skill.git
   ```
2. Point LLM at the skill folder in your project settings

## Prerequisites

You'll need a Visualping API key. Get one from your [Developer settings](https://visualping.io/account/developer) (up to 5 active keys).

## Usage examples

Once installed, just talk to LLMs naturally:

- _"Monitor https://example.com for changes"_
- _"List all my active monitors"_
- _"Show me the change history for job 12345"_
- _"Pause my monitor on google.com"_
- _"Set up a Slack notification for my monitor"_
- _"Create a monitor that checks every hour"_
- _"Create a monitor with my saved settings"_
- _"Bulk import these 50 URLs"_
- _"Run my monitor right now"_
- _"Show me recent activity across all my monitors"_
- _"Create a label called 'Competitors' and tag these jobs"_

LLM will ask for your API key if you haven't provided one, then either generate a code snippet or execute the call directly — your choice.

## API coverage

| Action                         | Endpoint                            |
| ------------------------------ | ----------------------------------- |
| Authenticate                   | `POST /v2/token`                    |
| Get account info               | `GET /describe-user`                |
| List jobs                      | `GET /v2/jobs`                      |
| Create job                     | `POST /v2/jobs`                     |
| Create job from saved settings | `POST /v2/jobs/from-saved-settings` |
| Bulk create jobs               | `PUT /v2/jobs/bulk`                 |
| Get job details & history      | `GET /v2/jobs/{jobId}`              |
| Update job                     | `PUT /v2/jobs/{jobId}`              |
| Delete job                     | `DELETE /v2/jobs/{jobId}`           |
| Activity feed (all jobs)       | `POST /v2/jobs/report-page`         |
| Run job immediately            | `POST /v2/jobs/launch`              |
| List labels                    | `GET /v2/jobs/labels`               |
| Create labels                  | `POST /v2/jobs/labels`              |
| Update labels                  | `PUT /v2/jobs/labels`               |
| Delete label                   | `DELETE /v2/jobs/labels`            |
| Assign/unassign labels on jobs | `PUT /v2/jobs/joblabels`            |

## Direct execution

The skill can execute API calls directly (instead of just generating snippets) when Claude has terminal access. This works out of the box in **Claude Code** and **Cowork**.

In **Claude.ai** with an organization account, your org owner needs to add these domains to the network allowlist:

- `account.api.visualping.io`
- `job.api.visualping.io`
- `api.visualping.io`

Without these domains, the skill still works — it generates ready-to-run code snippets you can copy and execute in your own environment.

## Defaults

When creating a monitor, the skill uses these defaults unless you specify otherwise:

| Setting             | Default          | Notes                                     |
| ------------------- | ---------------- | ----------------------------------------- |
| Check frequency     | Daily (1440 min) | Common: 5m, 15m, 30m, 1h, 6h, 12h, 1d, 1w |
| Active              | true             | Starts monitoring immediately             |

For preset-based creation (`POST /v2/jobs/from-saved-settings`), the saved settings determine defaults (including frequency) unless you explicitly override them.

## Support

- **API docs**: [visualping.io](https://visualping.io)
- **API support**: support@visualping.io
- **Pricing & plans**: [visualping.io/pricing](https://visualping.io/pricing)
