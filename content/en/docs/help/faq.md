---
title: "FAQ"
description: "Answers to frequently asked questions."
lead: "Answers to frequently asked questions."
date: 2020-10-06T08:49:31+00:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "help"
weight: 9630
toc: true
---

## Do I need PHP if I use the Deno backend?

Yes, but only for the generator. `setup.php` runs `puneetxp/compile-php`, which is written in PHP. The Deno app runs without PHP at run time.

## Which backend should I pick?

- **PHP** is the most complete. It runs anywhere that hosts PHP 8.2.
- **Deno** is faster, and it is what the INTAX app runs in production. It is still alpha.
- **Python (FastAPI)** runs on PostgreSQL, and the farming platform uses it in production. It is the newest backend.
- **.NET, Go and Spring** are not usable from the generator yet.

## Can I edit generated code?

You can edit controllers, as long as their key stays in `config.json` → `table`. Models, interfaces, routes and SQL are rewritten on every run, so change the schema rather than those files.

## Who is the admin?

In PHP, the user with `id = 1` is always `isuper`, and other users get roles through `active_roles`. In Deno, roles come from `active_roles`, and in `0.1.x` user 1 also bypasses role checks.

## Does it support PostgreSQL?

For SQL generation, yes. Set `"postgresql": true` and apply the files with `psql -f database/Migration.sql`. The PHP and Deno runtimes use MySQL.

## Contact the creator

- [Developer site](https://puneetxp.github.io)
- [Email](mailto:puneetsharma9@hotmail.com)
- [GitHub projects](https://github.com/puneetxp)
