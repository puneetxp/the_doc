---
title: "PHP"
description: "The PHP runtime puneetxp/the: routing, ORM, auth, sessions and helpers."
lead: "The PHP backend runs on `puneetxp/the` (repository `the_lib`), a dependency-free runtime for PHP 8.2+. The generator writes models and controllers against it."
date: 2026-09-28T10:00:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1000
toc: true
---

## Install

```bash
cd php && composer require puneetxp/the
```

The package namespace is `The\`. Generated app code lives in the `App\` namespace, under `php/App/`.

## Request flow

```text
.htaccess  ──►  php/index.php  ──►  new The\Route($route)  ──►  Controller::method(...$params)
```

```php
// php/index.php
require_once __DIR__ . '/env.php';            // define()s from config.json → env
require_once __DIR__ . '/additionalenv.php';  // runtime $_ENV, editable via /api/env
require_once __DIR__ . '/vendor/autoload.php';
require_once __DIR__ . '/Routes/web.php';     // compiled $route
new Route($route);
```

`The\Route` does the following:

1. Starts the session with the `secure`, `sslhost`, `httponly` and `samesite` constants.
2. Picks the routes for the request method. `$_POST['_method']` overrides the method, so HTML forms can send PATCH and DELETE.
3. Matches the path and runs the route's guards, login check and role check.
4. Calls the handler, passing each `.+` segment as an argument, then echoes the return value.

If the request carries `$_POST['_action']`, it holds a list of `{url, method, data}` sub-requests. The router runs them all and returns a single JSON map.

## Routes

Routes are plain nested arrays in `php/Routes/pre/`. `php php/set.php` compiles them with `The\compile\RouteCompile` into `php/Routes/web.php`. **Run it after every route change.**

```php
use The\Auth;
use App\Controller\Isuper\IsuperBrandController;

$routes = [
  ["method" => "POST", "path" => "/login",  "handler" => [Auth::class, "login"]],
  ["method" => "GET",  "path" => "/logout", "handler" => [Auth::class, "logout"]],
  [
    "path"    => "/isuper",
    "islogin" => true,
    "roles"   => ["isuper"],
    "child"   => [
      ["path" => "/brand", "crud" => ["class" => IsuperBrandController::class, "crud" => ["c","r","u","d","a"]]],
      ["path" => "/report/.+", "handler" => [ReportController::class, "show"]],   // show($id)
    ],
  ],
];
```

| Key | Meaning |
|---|---|
| `path` | A path segment. Use `.+` for a parameter; each one is passed to the handler as a positional argument. |
| `method` | `GET` (the default), `POST`, `PATCH`, `PUT` or `DELETE`. |
| `handler` | `[Class::class, "method"]`. |
| `islogin` | Requires a logged-in session. |
| `roles` | Requires the user to have at least one of these roles. |
| `guard` | A list of `[Class::class, "method"]` callables that run before the handler. |
| `child` | Nested routes. `path`, `islogin`, `roles` and `guard` are inherited. |
| `group` | Routes keyed by HTTP method: `["GET" => [...], "POST" => [...]]`. |
| `crud` | `["class" => Controller::class, "crud" => [letters]]`. Expands into the routes below. |

{{< alert icon="⚠️" text="<code>roles</code> and <code>guard</code> are only checked when <code>islogin</code> is true, whether it is set on the route itself or inherited from a parent." />}}

### CRUD shorthand

| Letter | Method and path | Handler |
|---|---|---|
| `a` | `GET /model` | `all()` |
| `r` | `GET /model/{id}` | `show($id)` |
| `c` | `POST /model` | `store()` |
| `w` | `POST /model/where` | `where()` |
| `u` | `PATCH /model/{id}` | `update($id)` |
| `p` | `PUT /model` | `upsert()` |
| `d` | `DELETE /model/{id}` | `delete($id)` |

### Generated routes

The generator writes one file per role in `php/Routes/pre/api/`: `Isuper.php`, `Islogin.php`, `Ipublic.php` and one per custom role. `Routes/pre/web.php` mounts them all under `/api`, next to the built-in auth routes:

| Route | Handler |
|---|---|
| `POST /api/login` with `{email, password, remember_me?}` | `Auth::login` |
| `GET /api/login` | `Auth::status` |
| `POST /api/register` with `{name, email, password}` | `Auth::register` |
| `GET /api/logout` | `Auth::logout` |
| `/api/env/*` (isuper) | Reads or edits `$_ENV` and saves it to `additionalenv.php`. |

## Controllers

Generated controllers are classes of static methods, one class per role and model:

```php
namespace App\Controller\Isuper;
use App\Model\Role;

class IsuperRoleController {
    public static function all() {
        if (isset($_GET["latest"])) {
            return Role::wherec([["updated_at", ">", $_GET["latest"]]])->get();
        }
        return Role::all();
    }
    public static function show($id)   { return Role::find($id); }
    public static function store()     { return Role::create($_POST)->getInserted(); }
    public static function update($id) { Role::where(["id" => [$id]])->update($_POST); return Role::find($id); }
    public static function upsert()    { return Role::upsert(json_decode($_POST["roles"]))->getsInserted(); }
}
```

Models convert to JSON when echoed, so a controller can return a model directly.

To customise a controller, edit it and **keep its key** in `config.json` → `table`, so it isn't overwritten. See [Config]({{< relref "config#protecting-hand-edited-controllers-table" >}}).

## Model

Generated models extend `The\Model` and declare `$table`, `$name`, `$model` (the columns), `$fillable`, `$nullable` and `$relations`.

```php
User::all();                                        // every row
User::find(5);                                      // by id, or null
User::find("a@b.c", "email");                       // by another column
User::where(["enable" => [1], "role" => ["a","b"]])->get();   // values are arrays → IN (...)
User::wherec([["created_at", ">", "2026-01-01"]])->get();     // custom operators
User::where(["enable" => [1]])->andWhereC([["id", ">", 10]])->orwhere(["id" => [1]])->get();
User::where(["id" => [5]])->first(["name", "email"]);
User::where(["enable" => [1]])->count();
User::all()->paginate(1, 25);                       // also reads ?page= & ?pageItems=
```

### Writing

```php
User::create(["name" => "A", "email" => "a@b.c"])->getInserted();
User::insert([[...], [...]]);
User::upsert([[...], [...]])->getsInserted();       // ON DUPLICATE KEY UPDATE
User::where(["id" => [5]])->update(["name" => "B"]);
(new User)->toggle(["id" => [5]], "enable");   // flips the column
User::delete(["id" => [5]]);
```

### Relations (eager loading)

```php
Invoice::find(1)->with(["client", "user"])->sort();
Book::all()->with([["invoice" => ["client"]]])->sort();   // nested
```

`with()` loads the related rows in one query per relation. `sort()` then nests them under each parent row. A relation declared as one-to-one becomes an object rather than a list. Children are ordered by their `sort` column when that column is fillable.

Use `The\DB::raw($sql, $bind)` for raw SQL.

## Auth and sessions

| Class | Methods |
|---|---|
| `The\Auth` | `login`, `register`, `status`, `logout`, `profile`, `profileupdate` |
| `The\Sessions` | `create($user)`, `roles()`, `get_current_user()` |
| `The\SocialAuth` | `g_auth($token)` (Google), `f_auth($token)` (Facebook) |

- Passwords are hashed with `sha3-256`.
- **The user with `id = 1` is always `isuper`.** Every other role comes from `active_roles` → `roles.name`.
- A successful login returns `{id, name, email, roles}`.

## Request and response helpers

```php
use The\{Req, Response};

$data = Req::only(["name", "email"]);    // filter $_POST
$one  = Req::one("name");

return Response::json($data);
return Response::not_found("Missing");       // 404
return Response::not_authorised("Nope");     // 403
return Response::unprocessable($errors);     // 422
return Response::bad_req("Bad");             // 400
return Response::NotLogin();                 // 401
```

## Files, images and mail

```php
use The\{FileAct, Img, Mail};

$saved = FileAct::init($_FILES["photo"])->public("brand")->fileupload($_FILES["photo"], "logo");
// → ["name" => …, "path" => …, "public" => "/storage/brand/logo.png"]

Img::webpImage(source: $saved["path"], destination: "…/logo.webp", x: 300, quality: 80);

(new Mail)->to(["a@b.c"])->subject("Hi")->message("<b>Hello</b>")->send();
```

## `env.php`

`env.php` is generated from `config.json` → `env`:

```php
define('host', "example.com");
define('sslhost', ".example.com");
define('dbhost', "localhost");
define('dbname', "demo");
define('dbuser', "username");
define('dbpwd', "password");
define('samesite', "Strict");
define('httponly', true);
define('secure', true);   // set false for plain-http local development
```
