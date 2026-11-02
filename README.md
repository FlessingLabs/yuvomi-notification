# Yuvomi: Telegram, Discord and a daily preview

[Deutsch](README.de.md) · **English**

Notifications from [Yuvomi](https://github.com/ulsklyc/yuvomi) via Telegram and Discord, with personal targets per member and a daily preview at a time each member chooses.

> **Where the code lives:** not here, but in the FlessingLabs fork of Yuvomi, branch [`feat/notifications`](https://github.com/FlessingLabs/yuvomi/tree/feat/notifications). A Yuvomi module runs only in the browser and cannot send at 7 pm when nobody has the app open. Bot tokens also must not sit in module storage, where anyone with module access could read them. Sending therefore belongs on the server, which already has Yuvomi's channel system with delivery tracking, retries and protection against internal addresses. This repository holds the guide and the proposal for the maintainer.

## What it does

- **Telegram and Discord** under *Settings → Messenger*. For Telegram you enter the bot token and the chat ID, for Discord the webhook address. Both are presets of the existing webhook channel, not a provider per service, which is the route Yuvomi itself takes (#692 in Yuvomi's changelog).
- **Tokens stay secret.** They are write-only, never shown again, and only enter the address when a message is sent (`{{token}}`).
- **Personal targets per member**, for example their own Telegram. An admin allows them per member. The member's reminders, private ones included, then also go there. Revoking the grant switches the member's targets off.
- **Daily preview**, set by each member for themselves: on or off, time, for today or tomorrow, content and targets. Content:
  - the day's events and tasks due, with exactly the member's own visibility;
  - birthdays and name days **from one week ahead**, then in every preview until the day;
  - waste collection;
  - open items on the shopping lists.
- **Family preview** to a family target, such as a Telegram group. It shows only what everyone may see; a private event never appears there, not even the author's own.
- Long messages are shortened to fit (Telegram 4096, Discord 2000 characters).

## Setup

Requirement: Yuvomi from the fork image `ghcr.io/flessinglabs/yuvomi:module-storage-…` from the build with **migration 276** (shown in the start-up log).

### Telegram

1. In Telegram, open **@BotFather**, send `/newbot`, pick a name. The answer contains the **bot token** (`123456789:AA…`).
2. Send your new bot one message (a bot may not write to anyone first).
3. Find the **chat ID**: open `https://api.telegram.org/bot<TOKEN>/getUpdates` in a browser; it is under `"chat":{"id": …}`.
   - **Group:** add the bot to the group, write something in the group, then `getUpdates` again. Group IDs are negative (`-100…`).
4. In Yuvomi: *Settings → Messenger → add target*, service **Telegram**, enter bot token and chat ID, **Send test**.

### Discord

1. In Discord: edit channel → **Integrations** → **Webhooks** → **New Webhook** → **Copy Webhook URL**.
2. In Yuvomi: *Settings → Messenger → add target*, service **Discord**, paste the address, **Send test**.

### Who may do what

| Who | Can |
|---|---|
| Admin | own targets, family targets, family preview, grants per member |
| Member with grant | own targets and own daily preview |
| Member without grant | none of this; web push and family targets work as before |

An admin sets the grant under *Settings → Messenger → Who may set up their own targets*. Nothing is derived from role or age.

### Good to know

- The preview arrives at the chosen time in the **household time zone**. If the server was down then, it still comes up to two hours later; otherwise it is skipped for that day.
- **Test now** saves the settings and sends the preview at once to the first selected target.
- Family targets receive every member's reminders, but only those everyone may see (as Yuvomi's household channels always did).

## Technical

- Migration 276: `notification_personal_access` (grant per member), `notification_digests` (preview per member and household).
- Webhook channel: `{{token}}` in the path of the address (value in `secrets.token`), `maxLength`, presets `preset: telegram | discord`.
- Routes: `/api/v1/notifications/personal…` (own), `/api/v1/notifications/access`, `/api/v1/notifications/digest` (admin). Documented in Yuvomi's OpenAPI.
- The preview reads with the same readers as the overview: calendar, tasks, birthdays, waste, shopping.

Proposal for the maintainer: [UPSTREAM-PROPOSAL.md](UPSTREAM-PROPOSAL.md).

## License

MIT, like Yuvomi. © 2026 FlessingLabs.
