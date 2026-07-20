# SRS Language Buddy — Bot specification

**Archetype:** education

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for language learners that uses spaced repetition to schedule vocabulary reviews. Users create word-translation flashcards with optional example sentences, rate their recall difficulty, and receive adaptive reminders. Sessions persist across interruptions and respect daily new-card limits while tracking learning progress and streaks.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual language learners
- non-technical users

## Success criteria

- Users complete daily study sessions with adaptive card scheduling
- Review reminders trigger consistent engagement
- Session state persists across interruptions

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Initialize bot, show onboarding, and set daily new-card limit
- **/study** (command, actor: user, command: /study) — Begin study session with due cards and new cards
- **Create Deck** (button, actor: user, callback: deck:create) — Initiate deck creation flow
  - inputs: deck title
  - outputs: deck created confirmation
- **Add Card** (button, actor: user, callback: card:add) — Start card creation wizard
  - inputs: word, translation, example sentence
  - outputs: card added confirmation
- **Начать повторение** (button, actor: user, callback: study:start_reminder) — Resume study session from daily reminder

## Flows

### onboarding
_Trigger:_ /start

1. Welcome message
2. Set daily new-card limit
3. Optional starter deck import

_Data touched:_ User

### study_session
_Trigger:_ /study or button

1. Show due cards first
2. Show new cards up to daily limit
3. Track progress with index/total counter

_Data touched:_ Study Session, Card

### card_rating
_Trigger:_ «снова»/«трудно»/«хорошо»/«легко» buttons

1. Update card's ease factor
2. Adjust next review interval
3. Advance to next card

_Data touched:_ Card

### deck_management
_Trigger:_ Deck management buttons

1. List decks
2. Edit/delete cards
3. Import starter decks

_Data touched:_ Deck, Card

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with private settings and progress
  - fields: telegram_id, daily_new_card_limit, reminder_time, language_preference
- **Deck** _(retention: persistent)_ — Private collection of language cards
  - fields: title, description, visibility, source_tag
- **Card** _(retention: persistent)_ — Word-translation pair with SRS metadata
  - fields: front, back, example_sentence, ease_factor, interval, due_date, status
- **Study Session** _(retention: session)_ — Interruptible study queue state
  - fields: current_card_index, interrupted_flag, session_start_time
- **Daily Plan** _(retention: persistent)_ — User-configured study parameters
  - fields: max_new_cards_per_day, notification_enabled

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Deck creation and management
- Card editing/deletion
- Daily new-card limit configuration
- Reminder time scheduling
- Starter deck import

## Notifications

- Daily review reminder with button
- Session completion message
- Progress milestone notifications

## Permissions & privacy

- All user data is private and never shared
- No public decks or group features
- Session state persists only for the user's account

## Edge cases

- No due cards in session → show friendly message
- Interrupted session resumes exactly
- User disables reminders → stop notifications

## Required tests

- End-to-end study session with rating → scheduling updates
- Session interruption → resume from same card
- Daily limit reached → no new cards shown

## Assumptions

- Default SRS parameters are reasonable for most learners
- Starter decks cover common beginner vocabulary
- Users prefer evening review sessions by default
