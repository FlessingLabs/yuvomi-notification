# Yuvomi: Telegram, Discord und Tagesvorschau

**Deutsch** · [English](README.md)

Benachrichtigungen aus [Yuvomi](https://github.com/ulsklyc/yuvomi) per Telegram und Discord, mit eigenen Zielen je Person und einer Tagesvorschau zur selbst gewählten Uhrzeit.

> **Wo der Code liegt:** nicht hier, sondern im FlessingLabs-Fork von Yuvomi, Branch [`feat/notifications`](https://github.com/FlessingLabs/yuvomi/tree/feat/notifications). Ein Yuvomi-Modul läuft nur im Browser und kann nicht um 19 Uhr senden, wenn niemand die App offen hat. Außerdem dürfen Bot-Tokens nicht im Modulspeicher liegen, wo jeder mit Modulzugriff sie lesen könnte. Der Versand gehört deshalb in den Server, und dort gibt es schon Yuvomis Kanalsystem mit Zustellbuchhaltung, Wiederholung und Schutz gegen interne Adressen. Dieses Repository enthält die Anleitung und den Vorschlag an den Maintainer.

## Was es kann

- **Telegram und Discord** unter *Einstellungen → Messenger*. Für Telegram trägst du Bot-Token und Chat-ID ein, für Discord die Webhook-Adresse. Beides sind Vorlagen des vorhandenen Webhook-Kanals, kein eigener Anbieter je Dienst. Das ist der Weg, den Yuvomi selbst vorsieht (#692 im Yuvomi-Changelog).
- **Tokens bleiben geheim.** Sie werden nur gespeichert, nie wieder angezeigt, und stehen erst beim Senden in der Adresse (`{{token}}`).
- **Eigene Ziele je Person**, zum Beispiel das eigene Telegram. Ein Admin gibt sie je Mitglied frei. Erinnerungen der Person, auch private, gehen dann auch dorthin. Nimmt der Admin die Freigabe zurück, werden ihre Ziele ausgeschaltet.
- **Tagesvorschau**, die jede Person für sich einstellt: an oder aus, Uhrzeit, für heute oder morgen, Inhalt und Ziele. Inhalt:
  - Termine des Tages und fällige Aufgaben, mit genau den Sichtrechten der Person;
  - Geburtstage und Namenstage **ab einer Woche vorher**, dann in jeder Vorschau bis zum Tag;
  - Müllabfuhr;
  - offene Einträge der Einkaufslisten.
- **Familien-Vorschau** in ein Familienziel, etwa eine Telegram-Gruppe. Sie zeigt nur, was alle sehen dürfen; ein privater Termin steht dort nie, auch nicht der eigene.
- Lange Nachrichten werden passend gekürzt (Telegram 4096, Discord 2000 Zeichen).

## Einrichten

Voraussetzung: Yuvomi aus dem Fork-Image `ghcr.io/flessinglabs/yuvomi:module-storage-…` ab dem Stand mit **Migration 276** (steht beim Start im Log).

### Telegram

1. In Telegram **@BotFather** öffnen, `/newbot` senden, Namen vergeben. Er antwortet mit dem **Bot-Token** (`123456789:AA…`).
2. Deinem neuen Bot einmal eine Nachricht schreiben (ein Bot darf niemanden zuerst anschreiben).
3. **Chat-ID** herausfinden: im Browser `https://api.telegram.org/bot<TOKEN>/getUpdates` öffnen; unter `"chat":{"id": …}` steht sie.
   - **Gruppe:** Bot in die Gruppe aufnehmen, in der Gruppe etwas schreiben, dann wieder `getUpdates`. Gruppen-IDs sind negativ (`-100…`).
4. In Yuvomi: *Einstellungen → Messenger → Ziel hinzufügen*, Dienst **Telegram**, Bot-Token und Chat-ID eintragen, **Test senden**.

### Discord

1. In Discord: Kanal bearbeiten → **Integrationen** → **Webhooks** → **Neuer Webhook** → **Webhook-URL kopieren**.
2. In Yuvomi: *Einstellungen → Messenger → Ziel hinzufügen*, Dienst **Discord**, Adresse einfügen, **Test senden**.

### Wer darf was

| Wer | Kann |
|---|---|
| Admin | eigene Ziele, Ziele der Familie, Familien-Vorschau, Freigaben je Mitglied |
| Mitglied mit Freigabe | eigene Ziele und eigene Tagesvorschau |
| Mitglied ohne Freigabe | nichts davon; Web-Push und Familienziele wirken wie bisher |

Die Freigabe setzt ein Admin unter *Einstellungen → Messenger → Wer eigene Ziele einrichten darf*. Nichts wird aus Rolle oder Alter abgeleitet.

### Gut zu wissen

- Die Vorschau kommt zur eingestellten Zeit in der **Zeitzone des Haushalts**. War der Server da aus, kommt sie bis zu zwei Stunden später noch, sonst entfällt sie für diesen Tag.
- **Jetzt testen** speichert die Einstellungen und schickt die Vorschau sofort an das erste gewählte Ziel.
- Familienziele bekommen die Erinnerungen aller Mitglieder, aber nur solche, die jeder sehen darf (so wie Yuvomis Haushaltskanäle schon immer).

## Technisch

- Migration 276: `notification_personal_access` (Freigabe je Mitglied), `notification_digests` (Vorschau je Person und Haushalt).
- Webhook-Kanal: `{{token}}` im Pfad der Adresse (Wert in `secrets.token`), `maxLength`, Vorlagen `preset: telegram | discord`.
- Routen: `/api/v1/notifications/personal…` (eigene), `/api/v1/notifications/access`, `/api/v1/notifications/digest` (Admin). Beschrieben in Yuvomis OpenAPI.
- Die Vorschau liest mit denselben Lesern wie die Übersicht: Kalender, Aufgaben, Geburtstage, Müll, Einkauf.

Vorschlag an den Maintainer: [UPSTREAM-PROPOSAL.md](UPSTREAM-PROPOSAL.md).

## Lizenz

MIT, wie Yuvomi. © 2026 FlessingLabs.
