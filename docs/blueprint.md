# Crypto Watcher Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking crypto prices with customizable alerts, price checks, and optional morning summaries. Users manage watchlists with price-threshold and percentage-move alerts, while the owner receives anonymized usage metrics and top alert reports.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual crypto watchers

## Success criteria

- users receive configured price alerts and summaries
- owner receives anonymized analytics report

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with watchlist management and settings
- **/price** (command, actor: user, command: /price) — Check current price of specified ticker or full watchlist
  - inputs: ticker symbol (optional)
  - outputs: price data with changes
- **Add Coin** (button, actor: user, callback: watchlist:add) — Open coin selection with common ticker buttons and free-text entry

## Flows

### Add Watchlist Entry
_Trigger:_ watchlist:add

1. show common coin buttons + 'Add ticker' free text
2. validate and add selected ticker
3. configure alert rules via inline prompts

_Data touched:_ user profile, watchlist entries

### Configure Alert
_Trigger:_ alert:configure

1. select alert type (price threshold/percent move)
2. set parameters via buttons/typed input
3. confirm and save rule

_Data touched:_ watchlist entries

### Morning Summary
_Trigger:_ schedule:summary

1. check user's enabled status
2. compile price data
3. send formatted summary

_Data touched:_ user profile, watchlist entries

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User-specific settings including timezone, quiet hours, summary time, and cooldown preferences
  - fields: telegram_user_id, timezone, quiet_hours_start, quiet_hours_end, summary_time, cooldown_minutes
- **watchlist_entry** _(retention: persistent)_ — Monitored crypto ticker with alert rules
  - fields: user_id, ticker, friendly_name, price_threshold_direction, price_threshold_value, percent_change_threshold, timeframe_hours, enabled
- **alert_log** _(retention: persistent)_ — Record of triggered alerts for analytics and cooldown enforcement
  - fields: timestamp, user_id_hash, ticker, rule_type, old_price, new_price, percent_change

## Integrations

- **Public Price Feed** (required) — Fetch current crypto prices with retry logic
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- daily analytics report with active user count and top 10 alert types
- anonymized user data aggregation for insights

## Notifications

- private price alerts with full change details
- optional daily summary at user-selected local time

## Permissions & privacy

- all user data stored privately with encryption at rest
- anonymized analytics only for aggregate metrics
- no third-party data sharing

## Edge cases

- unknown tickers show correction guidance
- price feed outages trigger silent retries
- quiet hours block alerts without queuing

## Required tests

- alert trigger flow with cooldown enforcement
- morning summary delivery with 24h change calculation
- error handling for invalid tickers

## Assumptions

- price feed API is available with retry logic
- users will manage their own watchlists without bulk imports
