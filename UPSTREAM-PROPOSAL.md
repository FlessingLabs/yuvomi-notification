# Proposal: personal targets and a daily preview for notification channels

*Draft for a discussion in ulsklyc/yuvomi. Not posted; to be posted by the fork owner if wanted.*

## Problem

Families use Telegram or Discord as their everyday messenger. Yuvomi can already reach both through the generic webhook channel with a payload template (#692), but:

1. **Only admins can create channels**, and only household channels. `notification_channels` has `scope = 'user'` and delivery honours it for private reminders, but no route creates such a channel. A member cannot get their own reminders in their own Telegram.
2. **Telegram's token sits in the URL** (`/bot<token>/sendMessage`), and so does Discord's (`/api/webhooks/<id>/<token>`). In the webhook channel that means the token lives in `config_json` in plain text and comes back with every channel list.
3. **There is no summary.** Reminders arrive one by one; there is no "what is tomorrow" message at a chosen time.

## Proposal (implemented in the FlessingLabs fork, branch `feat/notifications`)

Kept inside the existing channel system; no provider per service.

- **`{{token}}` in the webhook address.** If the address contains `{{token}}` (only allowed in the path), the value comes from the write-only `secrets.token` at send time, and no extra bearer header is sent. Validated on save, like the payload template.
- **Presets** `config.preset: 'telegram' | 'discord'` fill address, template and `maxLength` from two fields. It is still the webhook provider, with the same delivery tracking and SSRF guard. A pasted Discord URL is reduced to `<id>/<token>`.
- **`maxLength`** shortens the body at a line end (Telegram 4096, Discord 2000).
- **Personal targets** under `/api/v1/notifications/personal`: scope and owner always come from the session; another member's channel is a 404. An admin grants this per member (`notification_personal_access`, migration 276); nothing is derived from role (DECISIONS 1). Revoking switches the member's channels off; delivery itself does not ask the grant, so personal channels from before stay as they are.
- **Daily preview** (`notification_digests`): per member and once for the household; time in the household zone, today or tomorrow, sections events, tasks, birthdays (seven days ahead), waste, shopping. The member's preview reads with `visibilityWhere` and `resolvePermissions`, the household preview with the wall's `household` readers, and goes only to household channels. It respects the household module switches (`modulesLeftOut`, `birthdaysSwitchedOff`). Sent once a day, within two hours after the time, marked before sending.
- The admin channel list now shows household channels only.
- Settings page *Messenger*, texts in all 27 locales, OpenAPI for every route and reason, tests (`test:notification-messenger`), user removal covers both tables.

## Open questions for the maintainer

- Would you rather have the grant as a permission capability under Roles & permissions? It has no module to sit under today, which is why it is its own table.
- Should the digest reuse `GET /dashboard` with a date parameter instead of its own readers?
- Is the two-hour catch-up window right, or should a missed preview simply be skipped?
