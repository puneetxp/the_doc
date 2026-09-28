---
title: "Config"
description: "Every key in config.json and what the generator does with it."
lead: "`config.json` sits at the project root. The generator reads it on every run and rewrites its `table` map afterwards."
date: 2023-08-09T16:57:25+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 991
toc: true
---

## Full example

```json
{
  "alias": { "component": "c", "layout": "l" },
  "env": {
    "host": "example.com",
    "sslhost": ".example.com",
    "dbhost": "localhost",
    "dbname": "demo",
    "dbuser": "username",
    "dbpwd": "password",
    "samesite": "Strict",
    "httponly": true,
    "secure": true
  },
  "fresh": false,
  "postgresql": false,
  "back-end": ["php"],
  "front-end": ["angular"],
  "angular": { "outputPath": "../public_html/manager", "assets": [] },
  "table": { "user": true, "invoice": true }
}
```

## Keys

| Key | Read by | Effect |
|---|---|---|
| `env.dbhost` `dbname` `dbuser` `dbpwd` | migrate / sync, Deno | The database connection. For Deno these are also written to `deno/.env` (`DBHOST`, `DBNAME` and so on). |
| `env.host` | Deno | `HOST=` in `deno/.env`. |
| `env.*` (every key) | PHP | Each key becomes a `define('key', value);` in `php/env.php`. The runtime relies on `sslhost`, `httponly`, `samesite` and `secure` for the session cookie. |
| `fresh` | migrate | `true` drops and recreates the database. With `false`, statements that fail are skipped and logged as `[SKIP]`. |
| `postgresql` | write / migrate / sync | `true` emits PostgreSQL SQL instead of MySQL. |
| `back-end` | `config()` | One or more of `"php"`, `"deno"` or `"python"`. |
| `front-end` | `config()` | One or more of `"angular"` or `"solidjs"`. |
| `angular.outputPath` | Angular | Patched into `angular/angular.json` as the build output path. |
| `angular.assets` | Angular | `"src/storage"` is symlinked to `storage/public`. Any other asset is copied from `config/angular/<asset>`. |
| `alias` | view compiler | Tag aliases for `Resource/View`. With `component → c`, you can write `<t-c.card>`. |
| `table` | PHP controllers | The protected-controller map. See below. |

{{< alert icon="👉" text="Older configs also contain <code>controller</code>, <code>model</code>, <code>mysql</code>, <code>interface</code> and <code>public</code>. The current generator ignores them, so they are safe to keep or remove." />}}

## Protecting hand-edited controllers: `table`

After each run, `write()` saves `config.json` with a key under `table` for every model. The PHP generator **only writes a controller when the model has no key in `table`**. The check is `isset`, which is true even when the value is `false`:

```json
"table": {
  "invoice": true,   // protected: never regenerated
  "brand": false     // ALSO protected, because the key exists
}
```

- To keep your custom code, leave the key in place. Use `true` to make the intent obvious.
- To regenerate a controller from scratch, **delete its key**, then run `php setup.php` again.

Models, interfaces, SQL and frontend services are always regenerated. Keep custom logic in controllers, or in files the generator does not own.

## Backends and frontends that are not wired in

| Value | State |
|---|---|
| `"dotnet"`, `"golang"`, `"spring"` | The generator classes exist, but `config()` never calls them and their templates are missing. **Setting these does nothing today.** See [.NET]({{< relref "dotnet" >}}), [Go]({{< relref "golang" >}}) and [Spring]({{< relref "spring" >}}). |
| `"vuets"` | This is recognised, but the Vue generator never runs its `set()` step, so no files are produced. See [VueJS]({{< relref "vuejs" >}}). |
