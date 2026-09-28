---
title: "INTAX Billing"
description: "A GST billing and accounting app built with THE: Deno, SolidJS and MySQL."
lead: "INTAX (`the_billing`) is a GST billing and accounting app, and the most complete THE project. It shows how generated code, hand-written controllers and per-tenant security fit together."
date: 2026-09-28T10:00:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 100
toc: true
---

## What it does

- **Books:** each user can run several companies, called books. Every piece of accounting data belongs to a book.
- **Accounting:** accounts, journals and a ledger.
- **Billing:** customers and invoices, with CGST/SGST line items, payments and print views.
- **Servers:** hosting servers billed monthly, quarterly or yearly. An invoice can be generated from a server.
- **Assets and stock** for each book.
- **Admin console** for the `isuper` role: users, catalog, invoices and overview numbers.

## Stack

| Layer | Technology |
|---|---|
| Schemas | 45 models in `database/Model/*.json` |
| Generator | `puneetxp/compile-php`, run by `setup.php` |
| Backend | Deno with `the@0.0.2`, listening on port **9000** |
| Frontend | SolidJS, Tailwind and Vite, dev server on port **3000**. Routing uses `the-solid-router`. |
| Database | MySQL |
| Production | nginx serves `solidjs/dist` and proxies `/api/` to `:9000` |

## Repository layout

```text
the_billing/
├── setup.php                 runs the generator (Deno + SolidJS)
├── exampleconfig.json        copy to config.json
├── database/
│   ├── Model/*.json          schemas
│   ├── migrations/*.sql      hand-written migrations
│   └── structure.sql …       generated SQL
├── deno/
│   ├── index.ts              server bootstrap
│   ├── dep.ts                runtime imports (the@0.0.2)
│   └── App/{Model,Interface,Controller,Routes,Guard}
└── solidjs/src/
    ├── shared/Service/       generated ModelService layer
    ├── shared/api.ts         typed client for book-scoped endpoints
    └── _admin/               admin UI
```

## Generator setup

INTAX calls only the steps it needs, rather than `config()`:

```php
$setup = new setup(__DIR__);
$setup->table_set();
$setup->deno_set(param: "URLPatternResult");
$setup->solidjs_set();
$setup->write();
```

Its `config.json` has `table` entries for the controllers it has customised: `user`, `server`, `client`, `invoice` and `invoice_item`. The generator never overwrites controllers that are listed there.

## Routes

```ts
// deno/App/Routes/index.ts
const route_pre: _Routes = [
  { handler: Public.Home },
  { islogin: true, child: [...islogin, ...isuper, ...manager] },
  ...ipublic, ...webhook, ...Auth,
];
export const routes = compile_url_pattern(compile_routes(route_pre));
```

| Prefix | Who |
|---|---|
| `/login`, `/register`, `/logout`, `/profile` | Authentication |
| `/islogin/book` | The signed-in user's books |
| `/islogin/book/:book_id/{client,invoice,account,journal,journal_detail,asset,stock}` | Data that belongs to one book |
| `/islogin/server` | Servers, owned per user rather than per book |
| `/isuper/<model>` | The admin console. The admin role is **`isuper`**. |

The Deno routes have no `/api` prefix. In production, nginx removes it (`proxy_pass http://localhost:9000/;`, with a trailing slash), so the browser can call `/api/islogin/...`.

## Keeping users in their own books

Every book-scoped controller resolves `:book_id` through `ownedBook()` before it touches the database. The helper returns a 403 when the book belongs to someone else:

```ts
static async all(session: Session, param: URLPatternResult) {
  const book_id = await ownedBook(session, param);
  if (book_id instanceof Response) return book_id;
  const q = Client$.where({ book_id: [book_id] });
  const latest = latestParam(session);
  latest && q.andWhereC([["updated_at", ">", latest]]);
  return response.JSON((await q.get()).items, session);
}
```

Record-level lookups also filter on `book_id`, as in `where({ id: [id], book_id: [book_id] })`, so a user can't reach another tenant's rows by changing an id. See [Deno → per-tenant scoping]({{< relref "deno#pattern-per-tenant-scoping" >}}).

## Run it locally

```bash
# 1. dependencies
composer install
cp exampleconfig.json config.json        # fill in env.db*, keep the table flags

# 2. generate and migrate
php setup.php
mysql -u root -p intax < database/Migration.sql
mysql -u root -p intax < database/migrations/2026_09_26_billing_clients_invoices.sql

# 3. backend (reads deno/.env for DBHOST, DBUSER, DBPWD, DBNAME)
cd deno && deno run --watch --allow-all --unstable-kv index.ts

# 4. frontend
cd solidjs && npm install && npm run dev   # http://localhost:3000
```

{{< alert icon="⚠️" text="<code>deno/run.sh</code> hard-codes a Deno path from the production server (<code>/home/opc/.deno/bin/deno</code>). Run the <code>deno run</code> command above directly on your machine." />}}

The Deno routes have no `/api` prefix, so the Vite dev proxy strips it, just as nginx does in production:

```ts
// solidjs/vite.config.ts
proxy: {
  "/api": { target: "http://localhost:9000", changeOrigin: true, rewrite: (p) => p.replace(/^\/api/, "") },
},
```

## Production nginx

```nginx
server {
  server_name intax.in www.intax.in;
  root /var/www/intax/solidjs/dist;

  location / { try_files $uri $uri/ /index.html; }

  location /api/ {
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://localhost:9000/;     # trailing slash strips /api
    proxy_read_timeout 240s;
  }
}
```

## Lessons for your own app

1. **Know your runtime's bugs.** In `the@0.0.2` every user gets every role. INTAX recomputes the real roles after login (`withRealRoles()`). Upgrading to `@puneetxp/the@0.1.x` fixes the bug.
2. **Don't rely on the session's active book.** `session.ActiveLoginSession.book` is always the user's first book. Take the book from the URL and verify it with `ownedBook()`.
3. **Keep hand-written migrations separate.** Put changes the generator can't express in `database/migrations/`, and apply them after `Migration.sql`.
