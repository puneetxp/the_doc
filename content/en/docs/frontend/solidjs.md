---
title: "SolidJS"
description: "Generated SolidJS interfaces and services, the ModelService/Stores pattern, and IndexedDB delta sync."
lead: "With `\"solidjs\"` in `front-end`, the generator writes one TypeScript interface per model and a `Services.ts` file with one API service per model. The services run on a small `ModelService` base class that keeps a Solid store mirrored to IndexedDB."
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1020
toc: true
---

## Configure

```json
{ "front-end": ["solidjs"] }
```

```bash
npx degit solidjs/templates/ts solidjs
cd solidjs && npm install && npm install the-solid-router
cd .. && php setup.php
```

For development, point Vite at the backend:

```ts
// solidjs/vite.config.ts
server: { port: 3000, proxy: { "/api": { target: "http://localhost:9000", changeOrigin: true } } }
```

## Generated files

| Path | Contents |
|---|---|
| `src/shared/Interface/Model/<Name>.ts` | An interface matching the table. |
| `src/shared/Service/Services.ts` | One `ModelService` instance per model. |
| `src/shared/run.ts` | The `tables` list, plus start-up and shutdown hooks that hydrate or clear IndexedDB. |

```ts
// Services.ts (generated)
import { ModelService } from "./Service";
import { Client } from "../Interface/Model/Client";

export const ClientService = (new ModelService<Client>())
    .seTable("client")
    .seturl("/api/islogin/client/");

export * from "./Custom";   // lines like this are kept on regeneration
```

{{< alert icon="👉" text="Every service URL is set to <code>/api/islogin/&lt;model&gt;/</code>; <code>compile-php</code> 0.2.24 and earlier used <code>/islogin/&lt;model&gt;/</code>. For a model served under another role, override the URL, for example <code>ClientService.seturl(\"/api/isuper/client/\")</code>, or re-export a custom service from <code>Services.ts</code>." />}}

## The ModelService base class

**The generator does not write `ModelService` itself.** `Services.ts` imports it from `./ModelService` if that file exists, and from `./Service` otherwise, so you need to add one of those files to your project. A working implementation is in the INTAX billing app, at `solidjs/src/shared/Service/Service.ts` and `Store.ts`. Copy it from there.

`ModelService<T>` extends `Stores<T>`, which wraps a Solid `createStore({ data: [] })`. Every write to the store is also saved to IndexedDB.

| Method | HTTP call | Store |
|---|---|---|
| `all()` | On the first call it loads IndexedDB, then `GET url`. After that it sends `GET url?latest=<newest updated_at>`. | upsert |
| `get(id)` | `GET url/id` | upsert |
| `create(body)` | `POST url` | add |
| `update(id, body)` | `POST url/id` | update |
| `upsert(rows)` | `PATCH url` | upsert |
| `where(filter)` | `POST url/where` | upsert |
| `del(id)` | `DELETE url/id` | delete |
| `allstate()`, `findState()`, `getState()` | none (reads the store) | none |

```tsx
import { onMount, For } from "solid-js";
import { ClientService } from "../shared/Service/Services";

export default function Clients() {
  onMount(() => ClientService.all());
  return <For each={ClientService.allstate()}>{(c) => <p>{c.name}</p>}</For>;
}
```

{{< alert icon="👉" text="This base class sends <code>POST url/id</code> for update and <code>PATCH</code> for upsert. That matches Deno <code>the@0.0.2</code>. With the PHP runtime or Deno 0.1.x, change update to <code>PATCH</code> and upsert to <code>PUT</code>." />}}

## Sign-in and offline data

`run.ts` exports the `tables` list, which is also the list of IndexedDB object stores.

- **After login:** call `run.dbset()`. It hydrates every service from IndexedDB, so screens render immediately; the following `all()` calls then fetch only the changes.
- **On logout:** clear IndexedDB with `indexdb.The_clearData()`.

## Routing

The apps use [`the-solid-router`](https://www.npmjs.com/package/the-solid-router):

```tsx
import { Router, Routes, Route } from "the-solid-router";

<Router>
  <Routes>
    <Route path="/" guard={new run().set} component={Layout}>
      <Route path="login" guard={notLogin} component={LoginPage} />
      <Route path="book/:id" guard={isLogin} component={Book} />
    </Route>
  </Routes>
</Router>
```

Routes can be nested, and each one can take a `guard` that decides whether the route may be entered. INTAX uses `new run().set` as the guard on its root route: it hydrates the stores after login and clears IndexedDB after logout.

## Real-world example

The [INTAX billing app]({{< relref "intax-billing" >}}) is the most complete SolidJS client. It combines the generated `ModelService` layer with a hand-written typed client (`src/shared/api.ts`) for its book-scoped endpoints, which live under `/api/islogin/book/:book_id/…`.
