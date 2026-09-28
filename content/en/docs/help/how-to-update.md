---
title: "How to Update"
description: "Keep the generator, runtimes and UI packages up to date."
lead: "Update each package, then regenerate so the generated code matches the new versions."
date: 2020-11-12T13:26:54+01:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "help"
weight: 9100
toc: true
---

## Generator

```bash
composer update puneetxp/compile-php
php setup.php
```

Review the diff before you commit. Generated models, routes, interfaces and SQL can all change. Controllers with a key in `config.json` → `table` are left alone.

## PHP runtime

```bash
cd php && composer update puneetxp/the
```

Older templates pin `"puneetxp/the": "^0.1.0"`. Tags now run up to `0.1.304`, so check `php/composer.json`.

## Deno runtime

Bump the version in `deno/dep.ts`:

```ts
export { … } from "jsr:@puneetxp/the@0.1.16";
```

Moving from `deno.land/x/the@0.0.2` to JSR `0.1.x` changes the CRUD verbs, CORS, 404 handling and `Model.create`. Read [Version differences]({{< relref "deno#version-differences" >}}) first.

## Frontend

```bash
cd angular && npm update the-angular    # requires Angular 21
cd solidjs && npm update the-solid-router
```

## Database

After a schema change on a database that already holds data:

- use `$setup->sync()` to add missing columns and foreign keys; or
- write a migration under `database/migrations/`.

Don't run `migrate()` with `"fresh": true` against real data. It drops the database first.
