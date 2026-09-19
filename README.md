# shiba-uptime

Watches [shiba.pw](https://shiba.pw) from outside its own infrastructure and
sends a Telegram message when it cannot be reached.

## Why this is a separate, public repository

The site's source lives in a private repository. GitHub Actions gives a free
account 2,000 minutes a month on private repositories and unlimited minutes on
public ones — and a check running every ten minutes costs far more than 2,000
minutes a month. On a private repository this monitor would therefore run out
of budget partway through its first month and stop, silently. A monitor that
stops without saying so is worse than none at all, because its silence still
reads as "everything is fine".

So the probe lives here, on its own, where it is free to run forever. Nothing
in this repository is a secret: it contains a shell script and three URLs that
are already printed on the public internet. The Telegram credentials are
repository secrets, which are never readable — not in logs, not by anyone
browsing this page.

## Why it is not simply a health check on the server

Because that already existed, and it did not help. Twice the site was
unreachable for weeks while the checks running inside the container reported
themselves perfectly healthy — they were measuring the application, which was
fine, and not the path a visitor takes to reach it, which was not. A watchman
standing inside the building cannot tell you that it has burned down.

This runs on GitHub's machines and arrives the way a customer does: DNS,
Cloudflare, the tunnel, and only then the app.

## What it checks, every ten minutes

- `https://shiba.pw/up` answers 200
- the home page answers 200 after following its redirect to a locale
- `https://go.shiba.pw/vpZyOWOn` still redirects — a customer's QR code that is
  printed and hanging in their showroom, and cannot be reissued
- the TLS certificate has more than ten days left

Each failure is retried once, twenty seconds later, before anything is sent. A
single timeout is weather, not an outage, and an alarm that cries wolf gets
muted — which costs more than it saves.

## Setup

Two repository secrets, under *Settings → Secrets and variables → Actions*:

| Secret | Where it comes from |
| --- | --- |
| `TELEGRAM_BOT_TOKEN` | [@BotFather](https://t.me/botfather) → `/newbot` |
| `TELEGRAM_CHAT_ID` | message the bot once, then read `https://api.telegram.org/bot<TOKEN>/getUpdates` |

Run it by hand from the Actions tab (*uptime → Run workflow*) to confirm it
works before trusting it.
