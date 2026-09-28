---
title: "Quick Start"
description: "One page summary of how to start a new THE project."
lead: "From an empty folder to a generated, running API in a few commands."
date: 2020-11-16T13:59:39+01:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 110
toc: true
---

## Requirements

- [PHP 8.2+](https://www.php.net/downloads.php) and [Composer](https://getcomposer.org/download/). The generator is written in PHP, whichever backend you pick.
- MySQL or MariaDB. PostgreSQL is optional.
- [Node.js](https://nodejs.org/en) 18+ if you generate a frontend.
- [Deno](https://deno.com) 2 if you pick the Deno backend.

## 1. Create the project

```bash
composer create-project puneetxp/the_template_php my-app
cd my-app
```

The template contains `setup.php`, `config.json`, the auth models in `database/Model/` (`users`, `role`, `active_role`), and view and asset scaffolding under `Resource/`.

## 2. Configure

Edit `config.json`. Set the database credentials and choose what to generate:

```json
{
  "env": {
    "host": "localhost", "sslhost": "localhost",
    "dbhost": "localhost", "dbname": "my_app",
    "dbuser": "root", "dbpwd": "secret",
    "samesite": "Strict", "httponly": true, "secure": false
  },
  "fresh": true,
  "back-end": ["php"],
  "front-end": ["angular"],
  "table": {}
}
```

Accepted values are `back-end`: `php`, `deno` or `python`, and `front-end`: `angular` or `solidjs`. See [Config →]({{< relref "config" >}}) for every key.

{{< alert icon="⚠️" text="<code>fresh: true</code> drops and recreates the database when you migrate. Set it to <code>false</code> once you have real data." />}}

## 3. Describe your tables

Add one JSON file per table to `database/Model/`:

```text
database/
└── Model/
    ├── users.json
    ├── role.json
    ├── active_role.json
    └── brand.json        ← yours
```

```json
{
  "name": "brand",
  "table": "brands",
  "crud": { "isuper": ["c", "r", "u", "d", "a", "p"], "ipublic": ["r", "a"] },
  "enable": 1,
  "additional": ["slug"],
  "data": [{ "name": "name", "mysql_data": "varchar(255)", "datatype": "string" }],
  "relations": [{ "name": "photo", "default": "NULL" }]
}
```

See [Schema →]({{< relref "schema" >}}) for the full format.

## 4. Generate

```bash
php setup.php
```

This writes `database/*.sql` and the backend and frontend code. Apply the SQL to your database:

```bash
mysql -u root -p my_app < database/Migration.sql
```

You can also let the generator apply it by calling `$setup->migrate()` from `setup.php`. See [Generator →]({{< relref "setup" >}}).

## 5. Run

### PHP backend

```bash
cd php && composer install && cd ..
ln -s ../php/index.php public/index.php        # web root → front controller
ln -s ../storage/public public/storage         # uploads
php php/set.php                                # compile Routes/pre → Routes/web.php
php -S localhost:8000 -t public public/index.php   # built-in server ignores .htaccess, so route everything to index.php
```

### Deno backend

```bash
cd deno && deno run --watch --allow-all --unstable-kv index.ts   # serves :9000
```

### Frontend

```bash
cd angular && npm install && npx ng serve       # or: cd solidjs && npm install && npm run dev
```

Your API is now at `/api/ipublic/brand`, `/api/isuper/brand`, and so on. Log in with `POST /api/login` using `{email, password}`. The user with `id = 1` is always `isuper`.

## Next

- [Commands →]({{< relref "commands" >}}) lists the day-to-day scripts.
- [PHP →]({{< relref "php" >}}) or [Deno →]({{< relref "deno" >}}) explains how to customise generated controllers.
