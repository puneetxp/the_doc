---
title: "Troubleshooting"
description: "Known issues and their fixes."
lead: "Common problems, why they happen, and how to fix them."
date: 2020-11-12T15:22:20+01:00
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
menu:
  docs:
    parent: "help"
weight: 9620
toc: true
---

## Generator

### My controller changes were overwritten, or a controller won't regenerate

The PHP generator skips a controller whenever its model has **any** key under `config.json` → `table`, whether the value is `true` or `false`. After the first run, `write()` adds `false` for every model.

- **Keep your changes:** leave the key in place. Setting it to `true` makes the intent clearer.
- **Regenerate from scratch:** delete the key, then run `php setup.php`.

### A column I added doesn't appear in the SQL

Check the spelling of the schema key. It is `additional`, not `addtional`. Also make sure the column's object sits inside `data`.

### Inserts fail with "Field 'x' doesn't have a default value"

Columns are `NOT NULL` by default. Add `"default": "NULL"` to optional columns, or give them a `sql_attribute` such as `" NOT NULL DEFAULT 0"`.

### `array_keys(): Argument #1 must be of type array, null given` in `phpset.php`

In `compile-php` 0.2.24 and earlier, the PHP and Deno generators crash on a model that has no relations in either direction. Upgrade `compile-php`, or give the model a relation.

### "Warning: … relates to unknown table …, skipping"

A relation names a table that has no schema. Add the missing model, or fix the name.

### `"dotnet"`, `"golang"`, `"spring"` or `"vuets"` generates nothing

These targets are not wired into the generator yet. See [Config]({{< relref "config#backends-and-frontends-that-are-not-wired-in" >}}).

## PHP

### Protected `/api` routes are reachable without logging in

In projects created from `compile-php` 0.2.24 or earlier, the template's `Routes/pre/web.php` wraps the protected group in `"ilogin" => true`, which is a typo for `islogin`. Because `roles` and `guard` are only checked when `islogin` is true, **`/api/env` and every `/api/isuper` route are open to anyone**.

You can't just fix the key, because `Iauth.php` also holds `/login` and `/register`, which must stay public. Newer templates, and intaxing23, use this split:

1. Move `POST /login`, `GET /login`, `POST /register` and the social sign-in routes into `Routes/pre/api/Inotlogin.php` as `$inotlogin`.
2. Keep only `/logout` and `/auth/profile` in `$iauth`.
3. Mount them like this:

```php
require __DIR__ . "/api/Inotlogin.php";

["path" => "api", "child" => [
    ...$inotlogin,
    $ipublic,
    ["islogin" => true, "child" => [$isuper, ...$ienv, ...$iauth, $islogin]],
]]
```

Then run `php php/set.php`.

### `/api/auth/profile` shows or changes the wrong user

**This is a security issue in `puneetxp/the` up to 0.1.304.** `Auth::profile()` and `Auth::profileupdate()` filter on `user_id`, but `users` has no such column. `Model::where()` silently drops unknown columns, so the query runs **without a WHERE clause**:

- `GET /api/auth/profile` returns the first user in the table.
- `POST /api/auth/profile` updates **every** user row with the posted fields.

Until the library is fixed, remove the `/auth/profile` routes from `Routes/pre/api/Iauth.php`, or point them at your own controller that filters on `["id" => [$_SESSION['user_id']]]` and accepts only `name`, `phone` and `email`.

### A wrong password returns "User Not Found"

`Auth::login` builds a "Password Not Correct" response but doesn't return it, so execution falls through to the not-found reply. Treat both messages as "invalid credentials" in your UI.

### `/api/reset` returns a fatal error

In templates from 0.2.24 and earlier, the route points to `Auth::reseteverything`, which doesn't exist. Remove the route from `Routes/pre/api/Iauth.php`; newer templates no longer include it.

### Routes changed but nothing happens

Recompile the routes with `php php/set.php`. The runtime only reads `Routes/web.php`.

### Session cookie isn't set locally

`secure: true` stops the browser from sending the cookie over plain `http`. For local development, set `"secure": false` in `config.json` → `env` and regenerate.

## Deno

### `Deno.openKv is not a function`

Sessions are stored in Deno KV. Start the server with `--unstable-kv`.

### Every user can open admin routes

This is the role bug in `the@0.0.2`: `SessionRoles` grants every role. Upgrade to `jsr:@puneetxp/the@0.1.x`, or recompute the roles after login from `active_roles`.

### `new Router(...).route is not a function`

That API was removed. Use:

```ts
new Router(routes, req).URLPattern()?.run();
// with routes = compile_url_pattern(compile_routes(_routes))
```

Projects scaffolded by `compile-php` 0.2.24 or earlier have an `index.ts` that still uses the old call. Replace it. See [Deno]({{< relref "deno#server-bootstrap" >}}).

### Updates return 404

The update verb changed between releases. `the@0.0.2` expects `POST /:id`, while `0.1.x` expects `PATCH` or `PUT /:id`. Make sure your frontend service sends the verb your runtime expects.

### Deletes fail with "Unknown column 'deleted_at'"

In `compile-php` 0.2.24 and earlier, generated Deno `delete` handlers always soft-delete. Add `"additional": ["delete"]` to the model, or upgrade `compile-php`: newer versions only soft-delete when the column exists.

### Requests from the Vite dev server return "Not Found"

The Deno routes have no `/api` prefix. Add `rewrite: (p) => p.replace(/^\/api/, "")` to the Vite proxy entry, or configure your proxy to strip `/api`.

### Guards or roles don't run

They only run when `islogin` is true, either on the route or inherited from a parent route.
