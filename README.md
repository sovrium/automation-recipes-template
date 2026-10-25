# Kerlo Automations

> Automations that run in your own app, with every run kept.

Built with [Sovrium](https://sovrium.com) — a configuration-as-code interpreter: one config
file in, a complete self-hosted web application out.

## Use this template

Click **Use this template** on GitHub to copy this app into your own repository (clean
history, yours to modify), or scaffold it locally:

```bash
curl -fsSL https://sovrium.com/install | sh
sovrium init my-automation-recipes --template automation-recipes
```

## What's inside

A small studio's lead intake, automated by six recipes that run inside the app:

1. **Webhook → record** (`capture-lead-from-webhook`) — a POST carrying the bearer token
   becomes a lead. Without the token it is refused and writes nothing.
2. **Record → email + log** (`log-new-lead`) — every new lead emails the team and adds a
   line to the activity log. The email is tried three times, with a growing wait, before the
   run fails.
3. **Cron → digest** (`daily-digest`) — weekday evenings at 18:00, Paris time, one email to
   the team.
4. **Failure → alert** (`alert-on-failure`) — a run that failed after its last retry is
   written to the activity log with its error. Sovrium itself emails the operators; this
   recipe does not send a second email.
5. **Manual → sub-automation** (`review-recent-leads`) — the **Review latest leads** button
   on Activity. With no lead it stops at once, as a success; otherwise it calls recipe 6 and
   waits for its answer.
6. **Sub-automation** (`score-leads`) — started only by another automation: it loops over
   the ten latest leads and logs each with its tier through `log-lead-review`, one step
   declared once under `actions:` in `app.yaml` and placed with `$ref`.

Optional: a lead that arrives with a `message` gets a two-sentence summary written into it.
It is the `record-to-mistral-summary` recipe from the Sovrium library (`sovrium library add
recipe/record-to-mistral-summary`); set `MISTRAL_API_KEY` to turn it on — without it the app
runs exactly the same.

The pages:

- **Recipes** (`/`, public) — the first four recipes, each as its trigger, its steps and the lines
  of configuration that declare it, and a way to sign in.
- **Activity** — every run, newest first, with its recipe, its outcome and one line of why,
  and four figures above: runs in the log, succeeded, succeeded after a retry, failed after
  retries. `/activity?record=<id>` opens one run in a drawer with its attempts and its error.
- **Leads** — the leads the recipes wrote, newest first.

Sign-up is closed; an admin adds each account. Everything is declared in
[`app.yaml`](./app.yaml) and the [`config/`](./config) tree — no application code. Edit the
config, restart, done.

## What to try

Load the demo data first. It creates two demo accounts, eight leads and a week of runs,
dated relative to now:

```bash
SOVRIUM_SEED_PASSWORD=choose-a-password sovrium seed app.yaml
LEADS_WEBHOOK_TOKEN=choose-a-long-random-token sovrium start app.yaml
```

The week holds 17 runs: 15 succeeded, 1 succeeded after a retry (lead #7, Liam Carver) and 1
failed after 3 attempts (lead #6, Sofia Ricci: the mailbox rejected the sender).

1. Open `/` without signing in: the first four recipes and their declarations.
2. Sign in as **maelle.kerlo@kerlo.example** (admin) with the password you chose; you land
   on **Activity**. Find the Retried and the Failed run, and open the failed one with
   `/activity?record=12`: 3 attempts, and the error word for word.
3. Post a lead to the webhook, first without the token (refused, 401), then with it:

   ```bash
   curl -X POST http://localhost:3000/api/automations/capture-lead-from-webhook/webhook \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer choose-a-long-random-token' \
     -d '{"name": "Inès Carvalho", "email": "ines.carvalho@porto-design.example", "company": "Porto Design"}'
   ```

   The lead appears on **Leads**, and the run at the top of **Activity**.

4. Press **Review latest leads** on Activity: eight lines, one per lead, land at the top of
   the list, and the admin console's Runs page shows the two runs — the manual one and the
   one it called.
5. Sign in as **jules.morvan@kerlo.example** (member): he reads the runs and the leads, but
   only an admin deletes one. Both are demo accounts.

## Run locally

```bash
curl -fsSL https://sovrium.com/install | sh
cp .env.example .env
sovrium seed app.yaml        # needs SOVRIUM_SEED_PASSWORD in .env
sovrium start app.yaml --watch
```

Zero-config: embedded SQLite, local file storage, no variable required to boot.
[`.env.example`](./.env.example) lists the optional ones this app reads.
The first administrator comes from `AUTH_ADMIN_EMAIL` and `AUTH_ADMIN_PASSWORD` on the
first start, or from `sovrium admin create <email>`.
`--watch` reloads the app each time you save a config file.

## Deploy

Ship it to Sovrium Cloud from this directory: sign in once, then deploy. The CLI finds or
creates the app it deploys to, checks the variables it needs, bundles and uploads it, and
ends only once the app answers at its address.

```bash
sovrium login
sovrium deploy
```

Any other host works the same way: run the `sovrium` binary with this config (see the
[deployment guides](https://sovrium.com/en/docs/installation)).

> **The demo accounts are for trying the app, not for running it.** The seed gives every
> account it creates, the admin included, the one password in `SOVRIUM_SEED_PASSWORD`, and
> their addresses are in `seed/`. Before anyone else can reach the app, choose a long
> password or skip the seed, and change or delete the demo admin.

## About this repository

This repository is **auto-published from the
[Sovrium monorepo](https://github.com/sovrium/sovrium)** on every release
(source: [`src/templates/automation-recipes`](https://github.com/sovrium/sovrium/tree/main/src/templates/automation-recipes)).
Issues are welcome here; please send code contributions upstream to the monorepo so the
template stays in sync with the engine. The pinned engine release lives in
`.sovrium-version`.

License: [MIT](./LICENSE). The Sovrium engine itself is licensed separately
([BSL 1.1](https://github.com/sovrium/sovrium/blob/main/LICENSE.md)).
