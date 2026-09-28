---
title: "Commands"
description: "Day-to-day commands for THE projects."
lead: "The commands you will run most often, grouped by what they do."
date: 2020-10-13T15:21:01+02:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 120
toc: true
---

## Generator

| Command | What it does |
|---|---|
| `php setup.php` | Regenerates SQL, backend and frontend code from `database/Model/*.json` and `config.json`. |
| `php compile.php` | Compiles the HTML views in `Resource/View` into PHP page classes. These extend `\The\PageBase`. |
| `./realtime.sh` | Watches files with inotify. On a change it re-runs `compile.php` and `php/set.php`, runs sass watch, and refreshes the Composer autoload. |

## PHP backend

| Command | What it does |
|---|---|
| `cd php && composer install` | Installs the runtime, `puneetxp/the`. |
| `php php/set.php` | Compiles the route definitions in `php/Routes/pre/` into `php/Routes/web.php`. Run it after every route change. |
| `composer dump-autoload --working-dir=php` | Picks up new or renamed classes. |

## Deno backend

| Command | What it does |
|---|---|
| `deno run --watch --allow-all --unstable-kv index.ts` | Starts the API. `--unstable-kv` is required because sessions are stored in Deno KV. |
| `deno check App/Routes/index.ts` | Type-checks the whole route tree. |

## Frontend

| Command | What it does |
|---|---|
| `npm run dev` / `npx ng serve` | Starts the dev server for SolidJS (Vite) or Angular. |
| `npx vite build` / `npx ng build` | Builds for production. |

## Database

```bash
mysql -u <user> -p <db> < database/Migration.sql     # structure + relations + role inserts
```

`Migration.sql` combines `structure.sql`, `relation.sql` and `insert.sql`. With `"postgresql": true`, run `psql -f database/Migration.sql` instead.
