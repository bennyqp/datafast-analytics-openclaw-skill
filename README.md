# DataFast Analytics OpenClaw Skill

Query DataFast website analytics and visitor data from OpenClaw. This skill wraps the DataFast API and summarizes results for common analytics questions (overview, time series, realtime, breakdowns, visitors, goals, and payments).

## Requirements

- OpenClaw installed
- A DataFast API key

OpenClaw loads skills from `<workspace>/skills` with highest precedence.

## Configure API Key

Set the API key in `~/.openclaw/openclaw.json` so it is injected per run:

```json
{
  "skills": {
    "entries": {
      "datafast-analytics": {
        "enabled": true,
        "apiKey": "YOUR_DATAFAST_API_KEY",
        "env": {
          "DATAFAST_API_KEY": "YOUR_DATAFAST_API_KEY"
        }
      }
    }
  }
}
```

The skill requires `DATAFAST_API_KEY` to be present. OpenClaw will inject it into the process environment for each agent run.

## Usage

Ask OpenClaw analytics questions such as:

- "Show visitors and revenue for the last 30 days."
- "Visitors from x.com in the last 24 hours."
- "Realtime active visitors right now."
- "Top pages by visitors."
- "Create a custom goal for a visitor."

## What It Calls

Base URL: `https://datafa.st/api/v1/`

Common endpoints:

- `GET /analytics/overview`
- `GET /analytics/timeseries`
- `GET /analytics/realtime`
- `GET /analytics/realtime/map`
- `GET /analytics/devices`
- `GET /analytics/pages`
- `GET /analytics/campaigns`
- `GET /analytics/goals`
- `GET /analytics/referrers`
- `GET /analytics/countries`
- `GET /analytics/regions`
- `GET /analytics/cities`
- `GET /analytics/browsers`
- `GET /analytics/operating-systems`
- `GET /analytics/hostnames`
- `GET /visitors/{datafast_visitor_id}`
- `POST /goals`
- `DELETE /goals`
- `POST /payments`
- `DELETE /payments`

## Filters and Time Ranges

Use `startAt` and `endAt` together for time windows. For segmentation, use `filter_*` parameters supported by the DataFast API (example: `filter_referrer=is:x.com`). See the full API docs in `references/datafast-api-docs.md`.

## Security Notes

- Treat the API key as a secret. Do not paste it into chat prompts.
- OpenClaw injects skill env vars per run and restores the environment after the run ends.

## Development

Skill files:

- `/Users/benny/Desktop/datafast/skills/datafast-analytics/SKILL.md`
- `/Users/benny/Desktop/datafast/skills/datafast-analytics/references/datafast-api-docs.md`

If you want to publish to ClawHub, use `clawhub sync --all` from the repo root.
