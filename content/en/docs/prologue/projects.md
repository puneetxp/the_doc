---
title: "Projects"
description: "Every repository in the THE framework family: what it is, how it is published and how mature it is."
lead: "THE is spread across several repositories. This page lists each one: what it does, how it is published, and how far along it is."
date: 2026-09-28T10:00:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 105
toc: true
---

## How the pieces fit

```text
database/Model/*.json ─┐
config.json ───────────┼──► setup.php ──► puneetxp/compile-php (generator)
                       │                    │
                       │                    ├─► SQL (MySQL / PostgreSQL)
                       │                    ├─► Backend  : php/  deno/  python/
                       │                    └─► Frontend : angular/  solidjs/
                       │
Generated code runs on ─► puneetxp/the (PHP) · @puneetxp/the (Deno)
                         the-angular (Angular UI) · the-solid-router (SolidJS)
```

## Core

| Repository | What it is | Package | Status |
|---|---|---|---|
| [compile-php](https://github.com/puneetxp/compile-php) | The code generator. It reads the schemas and `config.json` and writes SQL, the backend and the frontend. It also contains the HTML-to-PHP view compiler. | Composer `puneetxp/compile-php` (latest tag `0.2.24`) | **Active.** This is the heart of the framework. |
| [the_lib](https://github.com/puneetxp/the_lib) | The PHP runtime: `Model`, `Route`, `Auth`, `Sessions`, `Response`, `FileAct` and more. | Composer `puneetxp/the` (latest tag `0.1.304`, PHP ≥ 8.2) | **Active** |
| [the_deno](https://github.com/puneetxp/the_deno) | The Deno runtime: router, session (on Deno KV), `Model` over mysql2, and response helpers. | JSR `@puneetxp/the` (`0.1.16`); older releases are on deno.land/x `the@0.0.2` | **Active**, still alpha |
| [the_template_php](https://github.com/puneetxp/the_template_php) | The starter template for new projects. It contains `setup.php`, `config.json`, the auth models, and the view and asset scaffolding. | Composer `puneetxp/the_template_php` (`type: template`) | Maintained. Its README is out of date. |

## Frontend

| Repository | What it is | Package | Status |
|---|---|---|---|
| [the-angular](https://github.com/puneetxp/the-angular) | Angular UI building blocks: `<the-form-dynamic>`, `<the-table-material>`, `<the-sidenav>`, login dialog, `IndexedDBService`, `AuthService`, NGXS login state and form helpers. | npm `the-angular` (`0.0.13`, Angular 21) | **Active** |
| [the-angular-material](https://github.com/puneetxp/the-angular-material) | The Angular 16 predecessor of `the-angular`. | none | **Legacy.** Superseded by `the-angular`. |
| the-solid-router | A small router for SolidJS, used by the SolidJS apps. | npm `the-solid-router` (`0.0.1`) | In use |
| [the_web_component](https://github.com/puneetxp/the_web_component) | Vanilla Custom Elements: `<editable-list>` and `<wysiwyg-bs>`. | none (a Vite demo) | **Prototype** |

## Other backends

| Repository | What it is | Status |
|---|---|---|
| [the_dotnet](https://github.com/puneetxp/the_dotnet) | A .NET 10 port of the runtime (`The.DotNet.Lib`): an ADO.NET `DB`, a full `Model` query layer, and ASP.NET Identity plus JWT `Auth`. | **Early.** Works as a project reference. It is not on NuGet. |
| the_dotnet_api | A sample ASP.NET Core Web API that uses `the_dotnet`. It only provides register and login endpoints. | Proof of concept. It is not a git repository. |
| [the_go](https://github.com/puneetxp/the_go) | Go helpers: `sqlbuilder`, `model`, `auth`, `session` and more. | **Skeleton.** It has no `go.mod`, and DB and auth are placeholders. |
| [the_spring](https://github.com/puneetxp/the_spring) | Java helpers in the `com.puneetxp.lib` package. | **Skeleton.** It has no `pom.xml`, and DB and auth are placeholders. |

The generator also ships a **Python (FastAPI)** backend. Its helper library lives in `compile-php/libraries_dev/the_python`. See [Python →]({{< relref "python" >}}).

## Apps and starters

| Repository | What it is |
|---|---|
| intaxing23 | The **intaxing.in** production app, running on PHP, Angular and MySQL. It is the reference PHP app. |
| apac-genaiacademy-c2 | A rural farming and livestock platform running on Python (FastAPI), SolidJS and PostgreSQL. It is the reference Python app, and the source of the `app/core` runtime. |
| [the_billing](https://github.com/puneetxp/the_billing) | **INTAX**, a GST billing and accounting app. It runs on Deno, SolidJS and MySQL, and is the reference real-world app. See [INTAX Billing →]({{< relref "intax-billing" >}}). |
| [the](https://github.com/puneetxp/the) | The original 2022 "start pack", with the generator scripts in `_setup/` and PHP, Angular, Vue and Solid samples. It is kept for history; new projects should start from `the_template_php`. |
| [thesolidmarket](https://github.com/puneetxp/thesolidmarket) | A 2022 marketplace demo pairing the PHP generator with SolidJS. It is the ancestor of today's SolidJS services. |
| [the_doc](https://github.com/puneetxp/the_doc) | This documentation site, built with Hugo and the Doks theme. |
