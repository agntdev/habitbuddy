# HabitBuddy — Bot specification

**Archetype:** education

**Voice:** motivational and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for habit tracking that provides scheduled reminders, progress metrics (streaks, completion percentages), and milestone notifications while maintaining strict user privacy. Users can add/edit habits with flexible scheduling options, mark completions with one-click buttons, and review weekly summaries with motivational feedback.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- People seeking to build or maintain habits
- Non-technical Telegram users valuing simplicity and privacy

## Success criteria

- Users can track 1–50 habits with consistent daily/weekly reminders
- Accurate streak tracking and progress metrics visible in weekly summaries
- All habit data remains private and accessible only to the owner

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with habit overview
- **Add new habit** (button, actor: user, callback: habit:add) — Initiate habit creation flow
  - inputs: habit name, frequency type, notification time
  - outputs: Habit creation confirmation, First reminder
- **Mark as Done** (button, actor: user, callback: action:done) — Record habit completion in notification or list view
  - inputs: habit ID, timestamp
  - outputs: Streak update, Progress indicator
- **View Weekly Summary** (button, actor: user, callback: report:week) — Display weekly progress metrics
  - inputs: user ID
  - outputs: Weekly completion percentage, Streak statistics

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Request optional name input
3. Confirm timezone (default from Telegram)
4. Prompt to add first habit

_Data touched:_ user_profile

### Habit Creation
_Trigger:_ habit:add

1. Request habit name
2. Select frequency type (daily/weekday/N-per-week)
3. Choose notification time
4. Add optional note/icon
5. Confirm habit creation

_Data touched:_ habit, user_profile

### Daily Reminder
_Trigger:_ scheduled_reminder

1. Send notification with 'Done' and 'Later' buttons
2. Handle 'Done' button click
3. Handle 'Later' button click (postpone or pause)

_Data touched:_ habit, habit_history

### Missed Day Handling
_Trigger:_ habit_missed

1. Detect unmarked day at 23:59 local time
2. Send gentle reminder with 'Mark Back' option
3. Process manual status override (missed/failure)

_Data touched:_ habit_history

### Weekly Report
_Trigger:_ report:week

1. Generate calendar view of habit completions
2. Calculate weekly statistics
3. Display motivational summary with progress

_Data touched:_ habit_history, user_profile

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **user_profile** _(retention: persistent)_ — User account and preferences
  - fields: telegram_id, name, timezone, notification_settings
- **habit** _(retention: persistent)_ — Defined habit with scheduling rules
  - fields: id, name, frequency_type, notification_time, status, note, icon
- **habit_history** _(retention: persistent)_ — Daily habit completion records
  - fields: habit_id, date, status, timestamp, manual_override

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/edit/delete habits
- Pause/resume habit tracking
- Adjust notification settings
- View weekly/monthly progress reports
- Change timezone manually

## Notifications

- Daily habit reminders with action buttons
- Weekly summary with progress metrics
- Milestone notifications (7/30/90/180/365 days)

## Permissions & privacy

- No third-party data sharing by default
- User data (Telegram ID, name, habits) is private and not shared

## Edge cases

- Timezone changes during habit tracking
- Missed habit markings after 23:59 local time
- Duplicate 'Done' button clicks on same day
- Manual backdating within 48-hour grace period

## Required tests

- End-to-end habit creation and first reminder flow
- Streak calculation after 7/30-day milestones
- Weekly summary display with 100% completion rate
- Timezone handling during DST transitions

## Assumptions

- Timezone defaults to Telegram client setting
- 48-hour backdating window prevents history manipulation
- N-per-week habits default to even distribution
- Milestone notifications use non-intrusive format
- One-click 'Done' is atomic operation (no duplicate counts)
