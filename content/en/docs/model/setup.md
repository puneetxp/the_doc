---
title: "Generator"
description: "How setup.php and puneetxp/compile-php turn schemas into SQL, backends and frontends."
lead: "`setup.php` is a few lines that call the `puneetxp/compile-php` generator. Run it after every schema change."
date: 2023-08-09T16:57:25+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 992
toc: true
---

## Install

The generator is a Composer library. The template already requires it:

```bash
composer require puneetxp/compile-php
```

## `setup.php`

```php
<?php
use Puneetxp\CompilePhp\setup;
require "./vendor/autoload.php";

$setup = new setup(__DIR__);   // reads ./config.json and ./database/Model/*.json
$setup->config();              // generate everything selected in config.json
```

`config()` runs `table_set()`, then every back-end and front-end listed in `config.json`, then `write()`. You can also choose the steps yourself. The [INTAX billing app]({{< relref "intax-billing" >}}) does this:

```php
$setup = new setup(__DIR__);
$setup->table_set();
$setup->deno_set(param: "URLPatternResult");   // handler param type for URLPattern routing
$setup->solidjs_set();
$setup->write();
// $setup->migrate();
```

### Methods

| Method | What it does |
|---|---|
| `table_set()` | Loads and normalises every schema. It adds the default columns, `enable`, the `additional` columns and reverse relations, and registers each model in the `table` map. |
| `php_set()` | Generates the PHP models, controllers and route files. |
| `deno_set($param)` | Generates the Deno models, interfaces, controllers and routes, plus `deno/.env`. |
| `python_set()` | Generates the FastAPI app. |
| `angular_set()` | Generates the Angular interfaces, services, NGXS state and form validation. |
| `solidjs_set()` | Generates the SolidJS interfaces, `Services.ts` and `run.ts`. |
| `write()` | Writes the SQL files and saves `config.json`. |
| `migrate()` | Runs `database/Migration.sql` against `env.db*`. If `fresh` is set, it first drops and recreates the database. |
| `migratealter()` | Applies the `database/Mysql/Alter/*.sql` files, which are generated from `database/Model/Additional/*.json`. |
| `sync()` | Compares the schemas with the live database and adds missing columns and foreign keys, without dropping anything. |

## What gets generated

### SQL

```text
database/
├── Mysql/
│   ├── Structure/<Name>.sql          CREATE TABLE
│   ├── Relations/<Name>_relation.sql ALTER TABLE … FOREIGN KEY
│   ├── Insert/Roles_insert.sql       INSERT INTO roles (isuper, custom roles…)
│   └── Alter/<Name>_alter.sql
├── structure.sql                     all tables
├── relation.sql                      all foreign keys
├── insert.sql                        seed rows
└── Migration.sql                     structure + relation + insert
```

With `"postgresql": true`, only the combined files are written, in PostgreSQL syntax.

### Backends

| Target | Output |
|---|---|
| PHP | `php/App/Model/<Name>.php`, `php/App/Controller/<Role>/<Role><Name>Controller.php`, `php/Routes/pre/api/<Role>.php` and `php/env.php`. The scaffold (`index.php`, `set.php`, `.htaccess`, `composer.json` and the auth controllers) is copied without overwriting existing files. |
| Deno | `deno/App/Model/<Name>.ts`, `deno/App/Interface/Model/<Name>.ts`, `deno/App/Controller/<Role>/<Name>Controller.ts`, `deno/App/Routes/<Role>.ts`, `deno/.env` and the `index.ts` scaffold (port 9000). |
| Python | `python/app/models`, `orm`, `services` and `api/<scope>/<model>/`, plus `python/app/api/routers.py` and `app/main.py`. |

### Frontends

| Target | Output |
|---|---|
| Angular | `angular/src/app/shared/Interface/Model`, `Service/Model/<Name>.service.ts`, `Ngxs/State`, `Ngxs/Action`, `Form/Validation`, `db/tables.ts` and `Service/run.service.ts`. It also patches `angular.json`. |
| SolidJS | `solidjs/src/shared/Interface/Model/<Name>.ts`, `Service/Services.ts` and `run.ts`. |

## Safe regeneration

Everything is regenerated on every run, **except PHP controllers that already have a key in `config.json` → `table`**. Put hand-written logic in those controllers. See [Config → table]({{< relref "config#protecting-hand-edited-controllers-table" >}}).

In `Services.ts`, the SolidJS generator keeps any `export * from "…"` lines you added. Use them to add custom services.

## View compiler

`compile-php` also compiles server-rendered HTML views into PHP classes:

```php
// compile.php
(new compilephp("Resource/View", __DIR__))->run();
```

Views under `Resource/View/{Layout,Component,Pages}` can use:

- component tags such as `<t-l.guest>` (with the aliases set in `config.json` → `alias`);
- `@props({...})`;
- `{$var}` output;
- `@foreach … @endforeach`.

The generated classes extend `\The\PageBase`.

## Uploads

The PHP runtime saves uploads under `storage/`. To expose them publicly, symlink them into the web root:

```bash
ln -s ../storage/public public/storage
```
