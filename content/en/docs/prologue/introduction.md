---
title: "Introduction"
description: "THE is a schema-first framework family: describe your tables once in JSON and generate SQL, a backend API and typed frontend code."
lead: "THE is a schema-first framework family. You describe each table once in JSON, and a generator produces the MySQL schema, a role-guarded REST backend and typed frontend services from it."
date: 2020-10-06T08:48:57+00:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

## How it works

Every THE project has the same three inputs and one command:

1. **Model schemas** in `database/Model/*.json`: one file per table, holding the columns, relations and which roles can do which CRUD operations. See [Schema →]({{< relref "schema" >}}).
2. **`config.json`** at the project root: database credentials plus the backends and frontends you want generated. See [Config →]({{< relref "config" >}}).
3. **`setup.php`**, which runs the generator [`puneetxp/compile-php`](https://github.com/puneetxp/compile-php). See [Generator →]({{< relref "setup" >}}).

```bash
php setup.php
```

The generator writes:

- **SQL**: `database/structure.sql`, `relation.sql`, `insert.sql` and `Migration.sql`, with MySQL as the default and PostgreSQL as an option.
- **A backend**: models, one controller per role and route files, for **PHP**, **Deno** or **Python (FastAPI)**.
- **A frontend layer**: TypeScript interfaces, API services and stores for **Angular** (NGXS) or **SolidJS**.

The generated code runs on a small runtime library for each language. The PHP runtime is [`puneetxp/the`]({{< relref "php" >}}) and the Deno runtime is [`@puneetxp/the`]({{< relref "deno" >}}). Both use the same concepts: a `Model` ORM with eager-loaded relations, a router with `islogin`, `roles` and `guard`, and a CRUD shorthand.

## Core concepts

| Concept | Meaning |
|---|---|
| **Model** | One JSON schema, which becomes one table, one model class and one TypeScript interface. |
| **Role namespace** | Each generated route lives under a role prefix. `isuper` is for admins, `islogin` is for any signed-in user and `ipublic` needs no auth. Custom roles such as `executive` are also supported. |
| **CRUD letters** | `c` create, `r` read one, `u` update, `d` delete, `a` read all, `w` where (filter), `p` upsert (bulk). These are set per role in the schema. |
| **Protected controller** | A controller you have edited by hand. Its key under `table` in `config.json` stops the generator from overwriting it. |

A schema such as this one:

```json
{
  "name": "client",
  "table": "clients",
  "crud": { "isuper": ["c", "r", "u", "d", "a"], "islogin": ["c", "r", "u", "a", "w"] },
  "data": [{ "name": "name", "mysql_data": "varchar(255)", "datatype": "string" }],
  "relations": ["user"]
}
```

produces:

- a `clients` table with a `user_id` foreign key;
- `/api/isuper/client` and `/api/islogin/client` endpoints for the listed operations;
- a `Client` TypeScript interface and a `ClientService` on the frontend.

## Where to go next

- [Projects →]({{< relref "projects" >}}) lists every repository in the family and its status.
- [Quick Start →]({{< relref "quick-start" >}}) takes you from zero to a running app.
- [INTAX Billing →]({{< relref "intax-billing" >}}) walks through a real production app built with THE, using Deno, SolidJS and MySQL.
