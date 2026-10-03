---
title: "Deno"
description: "The Deno runtime @puneetxp/the: router, sessions on Deno KV, the MySQL Model and responses."
lead: "`the_deno` is the Deno runtime, published on JSR as `@puneetxp/the`. It pairs a small, fast router with a MySQL `Model`, cookie sessions stored in Deno KV, and JSON response helpers."
date: 2023-08-09T16:57:25+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1010
toc: true
---

{{< alert icon="👉" text="The runtime is still alpha. Two lines are in use: <strong>JSR <code>@puneetxp/the@0.1.x</code></strong> (current) and <strong>deno.land/x <code>the@0.0.2</code></strong> (used by the INTAX app). They differ in CRUD verbs, CORS and error handling. See <a href=\"#version-differences\">Version differences</a>." />}}

## Install

Keep every import in one `dep.ts` so that bumping the version is a one-line change:

```ts
// deno/dep.ts
export {
  compile_routes, compile_url_pattern, hash, Model, response, Router, Session, setRole, DB,
} from "jsr:@puneetxp/the@0.1.16";
export type { _Routes, relation, Route_Group_with } from "jsr:@puneetxp/the@0.1.16";
```

The database settings come from the environment, or from `deno/.env`, which the generator writes:

```bash
DBHOST=localhost
DBUSER=root
DBPWD=secret
DBNAME=my_app
# 0.1.x only: DBPORT=3306  DBPOOL=4  DBSOCKET=/tmp/mysql.sock
```

## Server bootstrap

```ts
// deno/index.ts
import { Router, setRole } from "./dep.ts";
import { routes } from "./App/Routes/index.ts";
import { Role$ } from "./App/Model/Role.ts";

setRole((await Role$().all()).items);          // load the roles table once

Deno.serve({ port: 9000 }, async (req) =>
  await new Router(routes, req).URLPattern()?.run()
);
```

```ts
// deno/App/Routes/index.ts
import { _Routes, compile_routes, compile_url_pattern } from "../../dep.ts";

const route_pre: _Routes = [
  { handler: Public.Home },
  { islogin: true, child: [...islogin, ...isuper] },
  ...ipublic,
  ...Auth,
];
export const routes = compile_url_pattern(compile_routes(route_pre));
```

Run it with KV enabled, because sessions are stored in Deno KV:

```bash
deno run --watch --allow-all --unstable-kv index.ts
```

{{< alert icon="👉" text="The scaffold copied by <code>compile-php</code> after 0.2.24 already matches this. It pins <code>@puneetxp/the@0.1.16</code> and includes auth routes. The 0.2.24 scaffold and earlier pinned <code>the@0.0.0.4.8</code> and called the removed <code>.route(req)</code>; in projects created with those versions, replace <code>dep.ts</code> and <code>index.ts</code> by hand. The scaffold is only copied when a file doesn't exist yet." />}}

## Routes

```ts
export const isuper: _Routes = [{
  path: "isuper",
  roles: ["isuper"],
  child: [
    { path: "/client", crud: { class: IsuperClientController, crud: ["c", "r", "u", "d", "a", "w"] } },
    { path: "/report/:year", handler: ReportController.year },
    { path: "/stats", group: { GET: [{ handler: Stats.all }], POST: [{ path: "/refresh", handler: Stats.refresh }] } },
  ],
}];
```

| Field | Meaning |
|---|---|
| `path` | A URLPattern pathname, e.g. `/:id` or `/book/:book_id/client`. Slashes are normalised, so `"isuper"` works the same as `"/isuper"`. |
| `method` | Defaults to `GET`. |
| `handler` | `(session, param) => Promise<Response>`. |
| `islogin` | Requires a valid session cookie. The response is **401** otherwise. |
| `guard` | `((req) => Promise<false \| string>)[]`. Return `false` to allow the request; a string denies it and becomes the error message. |
| `roles` | The user needs at least one of these roles. |
| `child` | Nested routes. Paths are concatenated, and `islogin`, `guard` and `roles` are inherited. |
| `group` | Routes keyed by HTTP method. |
| `crud` | `{ class, crud: [letters] }`. See below. |

{{< alert icon="⚠️" text="<code>guard</code> and <code>roles</code> only run when <code>islogin</code> is true, either on the route or inherited from a parent. A public route with <code>roles</code> is <strong>not</strong> protected." />}}

### Handlers and parameters

Every handler has the same signature. Public routes get a `Session` too; it just has no login.

```ts
static async show(session: Session, param: URLPatternResult) {
  const id = param.pathname.groups.id;                                   // from "/:id"
  const latest = new URL(session.req.url).searchParams.get("latest");    // query string
  const body = await session.req.json();                                  // raw Request is session.req
  return response.JSON(await Client$().find(id), session);
}
```

### CRUD shorthand

| Letter | 0.1.x | 0.0.2 | Handler |
|---|---|---|---|
| `a` | `GET /` | `GET /` | `all` |
| `r` | `GET /:id` | `GET /:id` | `show` |
| `c` | `POST /` | `POST /` | `store` |
| `w` | `POST /where` | `POST /where` | `where` |
| `u` | `PATCH /:id` and `PUT /:id` | **`POST /:id`** | `update` |
| `p` | `PATCH /` and `PUT /` | `PATCH /` | `upsert` |
| `d` | `DELETE /:id`, plus `DELETE /perma_delete/:id` (isuper only) | `DELETE /:id` | `delete`, `perma_delete` |

Generated `delete` handlers soft-delete (set `deleted_at`) when the model has `"additional": ["delete"]`, and hard-delete otherwise. See [Schema]({{< relref "schema#extras-additional" >}}).

## Sessions and auth

When a route has `islogin`, the router:

1. reads the `PHPSESSID` cookie;
2. loads the session from Deno KV at `["users", id]` and checks that the user-agent matches;
3. runs the guards;
4. checks `roles` against `session.Login.roles`.

```ts
session.Login              // { id, name, email, roles: string[] }
session.ActiveLoginSession // { books, book, session_id, expire, ip, agent, … }
session.req                // the original Request
```

- **Log a user in:** `new Session(req).startnew(user, activeRoles, books)`, then return `response.JSONF(login, session.returnCookie())`.
- **Refresh the expiry:** `session.reactiveSession()`. `response.JSONS` also refreshes it.
- **Log out:** `session.removeSession()` and `session.removeCookie()`.
- **Cookie settings:** from the env keys `ssl` (domain), `samesite` and `secure`.
- **0.1.x only:** requests can also authenticate with `Authorization: Bearer <key>` against the `api_keys` table. The user with `id = 1` bypasses role checks.

{{< alert icon="⚠️" text="<strong>Bug in 0.0.2:</strong> <code>Session.SessionRoles</code> gives every user every role. Upgrade to 0.1.x, where it is fixed, or recompute the roles after login. INTAX does the latter in <code>withRealRoles()</code> in its <code>AuthController.ts</code>." />}}

## Model

Generated models extend `Model` and are exported as a **factory**, so each call starts a fresh query:

```ts
class Standard extends Model<Client> {
  constructor() {
    super("client", "clients", ["id"], ["name", "email", "book_id"], ["id", "name", "email", "book_id", "created_at", "updated_at"],
      { book: { table: "books", name: "book_id", key: "id", callback: () => Book$ } });
  }
}
export const Client$ = () => new Standard();
```

Code generated by older releases exports a shared instance instead (`export const Client$ = new Standard()`, called without `()`). That instance keeps its query state between calls, so prefer the factory form.

```ts
(await Client$().all()).items;                               // rows
(await Client$().find(5)).item;                              // one row (or undefined)
(await Client$().where({ book_id: [3], status: ["a", "b"] }).get()).items;   // IN (...)
Client$().where({ book_id: [3] }).andWhereC([["updated_at", ">", latest]]);  // custom operators
await Client$().create({ name: "Acme", book_id: 3 });        // 0.0.2: chain .getInserted()
await Client$().where({ id: [5] }).update({ name: "Acme Ltd" });
await Client$().upsert([{ id: 5, name: "…" }, { name: "new" }]);
await Client$().delete({ id: [5] });
await (await Invoice$().where({ id: [1] }).get()).with("client");   // eager-load a relation (async)
```

- Results are on `.item` for a single row and `.items` for a list.
- Writes are filtered through `fillable`.
- 0.1.x adds `softDelete`, `withJoin(rels, where)` (a SQL JOIN), `paginate`, `count`, `toJSON()`, `clone()` and an optional KV cache.

{{< alert icon="⚠️" text="<code>update()</code> without a <code>where()</code> updates <strong>every row</strong>. Always scope it." />}}

## Responses

```ts
response.JSON(body, session?, status?, headers?)   // JSON + refreshed session cookie
response.JSONS(body, session?, status?, headers?)  // same, and extends the session
response.JSONF(body, headers?, status?)            // JSON with explicit headers (e.g. a login cookie)
response.OPTIONS(req)                              // 0.1.x: 204 CORS pre-flight
```

In 0.1.x every helper adds CORS headers that echo the request's origin, with credentials allowed.

## Pattern: per-tenant scoping

A signed-in user should only reach their own rows.

### Generated from the schema

Write `"islogin": { "can": [...], "under": "book" }` in the model and `"islogin": { "can": [...], "owner": "user_id" }` in `book.json` (see [Which rows]({{< relref "schema#which-rows-owner-and-under" >}})). The generator then nests the route under `/book/:book_id/` and every method starts with the same check:

```ts
// App/Controller/Islogin/ClientController.ts (generated)
static async all(session: Session, param: URLPatternResult): Promise<Response> {
   const book_id = await ownedParent(session, param, Book$, "book_id", "user_id");
   if (book_id instanceof Response) return book_id;          // 404: not your book
   const rows = await Client$().where({ book_id: [book_id] }).get();
   return response.JSON(rows.items, session);
}

// App/scope.ts (template, copied once)
export async function ownedParent(session, param, parent, key, ownerColumn) {
  const id = Number(param.pathname.groups[key]);
  if (!Number.isInteger(id) || id <= 0) return response.JSON("Not Found", session, 404);
  const row = await parent().where({ id: [id], [ownerColumn]: [session.Login.id] }).first();
  if (!row) return response.JSON("Not Found", session, 404);
  return id;
}
```

The generated code calls models as factories (`Book$()`), as the current generator writes them. Scoped controllers are only generated with `param: "URLPatternResult"`.

### Hand-written (INTAX today)

INTAX nests the tenant id in the URL and checks ownership before every query:

```ts
// App/Controller/Islogin/_book.ts
export async function ownedBook(session: Session, param: URLPatternResult): Promise<number | Response> {
  const book_id = Number(param.pathname.groups.book_id);
  if (!book_id) return response.JSON("Book is required", session, 400);
  const book = (await Book$.find(book_id)).item;
  if (!book || book.user_id != session.Login.id) return response.JSON("Not Your Book", session, 403);
  return book_id;
}

// in a controller
const book_id = await ownedBook(session, param);
if (book_id instanceof Response) return book_id;
const rows = (await Client$.where({ book_id: [book_id] }).get()).items;
```

## Version differences

| | `the@0.0.2` (deno.land/x) | `@puneetxp/the@0.1.x` (JSR) |
|---|---|---|
| Update verb | `POST /:id` | `PATCH` or `PUT /:id` |
| Upsert verb | `PATCH` | `PATCH` or `PUT` |
| Permanent delete | none | `DELETE /perma_delete/:id` (isuper) |
| Route match | the last matching route wins | the first matching route wins |
| Not found | "Not Found" with status 200 | 404, and a 500 on exceptions |
| Roles | bug: everyone gets every role | fixed |
| CORS | none | on every response, plus `OPTIONS` |
| API keys | none | `Authorization: Bearer` |
| `Model.create` | chain `.getInserted()` | sets `.item` |

## Benchmark

This is a simple JSON route measured with `wrk -t2 -c10 -d10s`, on an early release:

| | Requests/sec | Avg latency |
|---|---|---|
| THE router | 28,800 | 398 µs |
| Oak | 17,403 | 633 µs |

That makes it roughly 1.6× Oak's throughput on this micro-benchmark.
