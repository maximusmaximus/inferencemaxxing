# inferencemaxxing

Generalized quota-window calendar.

Point YAML packs at any inference product. The engine turns them into:

- an `.ics` feed Google Calendar can subscribe to
- Telegram reminders
- a local week-view web UI with product deep links

This public repo is the **engine + example packs**. A private instance repo holds the live calendar id, Telegram bot, and any product-specific clocks.

## Install

```bash
python -m pip install -e .
cp .env.example .env
```

## Commands

```bash
# write a calendar file
python -m inferencemaxxing --packs examples/packs ics --out quota.ics

# print the next 24 hours
python -m inferencemaxxing --packs examples/packs today

# local week view + ICS at /quota.ics
python -m inferencemaxxing --packs examples/packs serve --port 8787

# Telegram (needs TELEGRAM_BOT_TOKEN and a /start so we can learn chat id)
python -m inferencemaxxing --packs examples/packs notify --within 90
```

## Hook it to Google Calendar

The secret iCal URL Google shows you is **read-only**. This engine publishes its own feed. Two ways to get events onto a Google calendar:

1. **Subscribe (recommended)**  
   Run `serve` or host `quota.ics`. In Google Calendar: **Settings → Add calendar → From URL** → `https://YOURHOST/quota.ics`.

2. **Import into an existing calendar**  
   **Settings → Import & export → Import** the generated `quota.ics` and pick the target calendar.

Writing *into* an existing calendar without import requires Google Calendar API OAuth (`GOOGLE_CREDENTIALS_JSON` + `GOOGLE_CALENDAR_ID`). That is optional.

## Pack format

See [`examples/packs/_template.yaml`](examples/packs/_template.yaml).

```yaml
id: example
title: Example product
timezone: America/Los_Angeles
color: "#5B8DEF"
location: https://example.com
windows:
  - id: window-open
    title: Window OPEN
    kind: interval        # interval | daily | weekly
    every: 5h
    duration: 40m
    remind_minutes: [15]
```

Example packs in this repo:

- `examples/packs/google-ultra.yaml` — 5-hour Antigravity / Gemini / Flow windows
- `examples/packs/grok-heavy.yaml` — weekly Heavy / Build / Imagine / Voice / Bot

Copy `_template.yaml` for Cursor, Claude, Venice, or anything else with a refresh clock.

## Telegram

Create a bot with BotFather. Put the token in `.env` (never commit it). Open the bot and send `/start` once so the engine can store `TELEGRAM_CHAT_ID`.

Commands the bot advertises: `/start` `/today` `/next` `/week` `/links`.

## Security

Do not commit:

- Telegram bot tokens
- Google private iCal URLs (`/private-…/basic.ics`)
- OAuth client secrets

The public calendar id (`…@group.calendar.google.com`) is not a secret. The `/private-…/` token is.
