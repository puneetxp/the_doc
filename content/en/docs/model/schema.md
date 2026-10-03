---
title: "Schema"
description: "The model JSON format: columns, relations, role-based CRUD and extras."
lead: "Each file in `database/Model/` describes one table. The file sets the columns, the relations, and which roles may perform which operations."
date: 2023-08-09T16:57:25+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 990
toc: true
---

## Name and table

```json
"name": "brand",     // model name: singular, lowercase
"table": "brands"    // SQL table name: plural
```

`name` becomes the class name (`Brand`), the controller name (`IsuperBrandController`), the TypeScript interface and the URL segment (`/api/isuper/brand`).

## Default columns

Every table gets `id`, `created_at` and `updated_at` unless you override `default`:

| Column | SQL |
|---|---|
| `id` | `bigint UNSIGNED PRIMARY KEY AUTO_INCREMENT` |
| `created_at` | `TIMESTAMP DEFAULT CURRENT_TIMESTAMP` |
| `updated_at` | `TIMESTAMP … ON UPDATE CURRENT_TIMESTAMP` |

```json
"default": ["id", "updated_at"]   // e.g. a pivot table without created_at
```

`updated_at` also drives **delta sync**. Every generated `all` endpoint accepts `?latest=<timestamp>` and returns only the rows that changed after it. The Angular and SolidJS services use this for incremental refreshes.

## Columns: `data`

```json
"data": [
  { "name": "name",   "mysql_data": "varchar(255)",  "datatype": "string" },
  { "name": "email",  "mysql_data": "varchar(255) UNIQUE", "datatype": "string" },
  { "name": "note",   "mysql_data": "longtext",      "datatype": "string", "default": "NULL" },
  { "name": "status", "mysql_data": "varchar(20)",   "datatype": "string", "sql_attribute": " NOT NULL DEFAULT 'draft'" },
  { "name": "secret", "mysql_data": "varchar(255)",  "datatype": "string", "fillable": "false" }
]
```

| Field | Meaning |
|---|---|
| `name` | The column name. |
| `mysql_data` | The SQL type, copied verbatim. You can add inline modifiers such as `UNIQUE`. |
| `datatype` | The TypeScript type for the generated interfaces. Use `string` for varchar, text and date strings. Use `number` for int, decimal, float and tinyint. `boolean` and `Date` are also allowed. `json` becomes `any`, `array` becomes `string[]` and `vector` becomes `number[]`. |
| `default` | The default value. `"NULL"` makes the column nullable. |
| `sql_attribute` | Raw SQL appended after the type, e.g. `" NOT NULL DEFAULT 18"`. |
| `fillable` | Set it to `"false"` to keep the column out of mass assignment (`fillable`). |

{{< alert icon="⚠️" text="Columns are <strong>NOT NULL by default</strong>. When a column has neither <code>default</code> nor <code>sql_attribute</code>, inserts without that column fail. Mark optional columns with <code>\"default\": \"NULL\"</code>." />}}

## Enable flag

```json
"enable": 1
```

Adding `enable` with any value creates `enable TINYINT(1) DEFAULT 1`. The runtimes use it for toggling (`Model::toggle`), and admin UIs show it as a switch.

## Extras: `additional`

```json
"additional": ["slug", "seo", "delete"]
```

| Value | Adds |
|---|---|
| `slug` | `slug VARCHAR(255) NOT NULL` |
| `seo` | `title VARCHAR(255)` and `seo_description longtext`, both nullable |
| `delete` | `deleted_at TIMESTAMP NULL`, for soft deletes. The Deno `delete` handler sets it, and `perma_delete` removes the row. |

{{< alert icon="👉" text="<strong>Deno:</strong> the generated <code>delete</code> handler soft-deletes (sets <code>deleted_at</code>) when the model has <code>\"additional\": [\"delete\"]</code>, and hard-deletes otherwise. <code>compile-php</code> 0.2.24 and earlier always soft-deleted, so on those versions every model that grants <code>d</code> needs <code>\"delete\"</code>." />}}

{{< alert icon="👉" text="The key is spelled <code>additional</code>. Older docs used <code>addtional</code>, and the generator silently ignores that spelling." />}}

## Unique keys

```json
"unique": ["email", ["book_id", "invoice_number"]]
```

Each entry becomes `UNIQUE KEY <name>_<cols>_unique`. A nested list creates a composite key.

## Relations

List the models this table belongs to. Use `relations`; the older key `relation` works the same way.

```json
"relations": [
  "user",                                     // user_id → users.id  (NOT NULL)
  { "name": "photo", "default": "NULL" },     // photo_id, nullable
  { "name": "user", "alias": "owner_id" }     // owner_id → users.id, relation key "owner"
]
```

For a foreign key to a table under a different column name, use the explicit keyed form:

```json
"relations": {
  "buyer": { "name": "buyer_id", "table": "users", "key": "id" }
}
```

For every relation, the generator:

- adds a `<rel>_id bigint UNSIGNED` column and a `<model>_<rel>_id_foreign` foreign key;
- registers the relation on the model, so `->with('user')` / `.with('user')` eager-loads it;
- adds the **reverse** relation to the target model automatically, so `users` gets `brand`.

If a relation points to a table that doesn't exist, a warning is printed and the relation is skipped.

## CRUD and roles

`crud` maps a **role namespace** to the operations that role may perform:

```json
"crud": {
  "isuper":  ["c", "r", "u", "d", "a", "p", "w"],
  "islogin": ["c", "r", "u", "a", "w"],
  "ipublic": ["r", "a"],
  "roles": {
    "executive": ["r", "a", "u"]
  }
}
```

| Namespace | Who | URL prefix |
|---|---|---|
| `isuper` | Administrators | `/api/isuper/<model>` |
| `islogin` | Any signed-in user | `/api/islogin/<model>` |
| `ipublic` | Everyone | `/api/ipublic/<model>` |
| `roles.<name>` | Users holding that role in `active_roles` | `/api/<name>/<model>` |

The custom role names are inserted into the `roles` table for you.

| Letter | Operation | Handler |
|---|---|---|
| `a` | Read all (supports `?latest=`) | `all` |
| `r` | Read one | `show` |
| `c` | Create | `store` |
| `u` | Update | `update` |
| `d` | Delete | `delete` |
| `w` | Filter with a JSON body | `where` |
| `p` | Upsert many rows | `upsert` |

The HTTP verb for each letter depends on the runtime. See [PHP]({{< relref "php#crud-shorthand" >}}) and [Deno]({{< relref "deno#crud-shorthand" >}}).

## Which rows: `owner` and `under`

A list of letters says **what** a namespace can do, not **which rows**. With a plain list, `islogin` reads and writes every row in the table, exactly like `isuper`. To limit rows, write the entry as an object: `can` holds the letters, plus `owner` or `under`.

```json
// book.json: a user owns their books; an executive is assigned books
"crud": {
  "isuper":  ["c", "r", "u", "d", "a", "p", "w"],
  "islogin": { "can": ["c", "r", "u", "d", "a", "w"], "owner": "user_id" },
  "roles": {
    "executive": { "can": ["r", "a"], "owner": "executive_id" }
  }
}

// client.json: a client lives inside a book
"crud": {
  "isuper":  ["c", "r", "u", "d", "a", "p", "w"],
  "islogin": { "can": ["c", "r", "u", "d", "a", "p", "w"], "under": "book" },
  "roles": {
    "executive": { "can": ["r", "a", "w"], "under": "book" }
  }
}
```

| Entry | A row is yours when | URL |
|---|---|---|
| `["r", "a"]` | always (no filter) | `/islogin/client` |
| `{ "can": [...], "owner": "user_id" }` | its `user_id` is your login id | `/islogin/book` |
| `{ "can": [...], "under": "book" }` | its book is yours | `/islogin/book/:book_id/client` |

Where each value comes from, for `client.json` `"islogin": { "under": "book" }`:

- `book` and `book_id`: from `under`. The parent model is `book`; the URL part and the column are `book_id` (the same `<name>_id` rule as relations).
- `user_id`: from **`book.json`**, its `islogin` entry's `owner`. A role reads the parent's entry for the **same role**, so the executive route checks `executive_id` and the user route checks `user_id`. One table can have a different owner for each namespace.
- Your login id: from the session at request time.

On every request under a book, the generated controller first runs one query, `books WHERE id = :book_id AND user_id = <you>`. No row means **404** and nothing else runs. Then it filters by `book_id`, and on create, update and upsert it sets `book_id` from the URL, so the request body cannot pick or change the book.

Generation stops with a message when:

- the parent has no `owner` for that namespace (`client.json "islogin" says "under": "book", but book.json has no "owner" for "islogin".`);
- the parent schema does not exist;
- `public` / `ipublic` uses `owner` or `under` (there is no logged-in user).

`owner` is one column, so it means one person per row per namespace. Several people per row (for example a manager assigned to many clients) needs a link table and is not supported yet.

{{< alert icon="👉" text="Supported by the Deno backend and SolidJS frontend generators. The other generators read only the letters in <code>can</code>, so an object entry gives them the same unscoped code as a list; scope those by hand (Python: <code>app/core/ownership.py</code>)." />}}

## Photo models: `type`

```json
"type": { "name": "photo", "version": { "thumb": { "width": 300, "quality": 80 } } }
```

With this set, the PHP generator emits an upload controller instead of a plain CRUD one. It stores the original file and creates a webp version for each entry in `version`.

## Full example

This is `invoice.json` from the [INTAX billing app]({{< relref "intax-billing" >}}):

```json
{
  "name": "invoice",
  "table": "invoices",
  "crud": {
    "islogin": ["c", "r", "u", "d", "a", "w"],
    "isuper":  ["c", "r", "u", "a", "w"]
  },
  "data": [
    { "name": "invoice_number", "mysql_data": "varchar(255)",  "datatype": "string" },
    { "name": "amount",         "mysql_data": "decimal(12,2)", "datatype": "number" },
    { "name": "status",         "mysql_data": "varchar(255)",  "datatype": "string", "sql_attribute": " NOT NULL DEFAULT 'draft'" },
    { "name": "gst_rate",       "mysql_data": "float(8,2)",    "datatype": "number", "sql_attribute": " NOT NULL DEFAULT 18" },
    { "name": "due_date",       "mysql_data": "varchar(50)",   "datatype": "string" },
    { "name": "notes",          "mysql_data": "varchar(255)",  "datatype": "string", "default": "NULL" }
  ],
  "relation": ["book", "user", { "name": "client", "default": "NULL" }]
}
```

## Legacy format

Projects from 2022 (`the`, `thesolidmarket`) used a flat `"crud": ["c","r","u","d"]` array with a separate `"roles": {"read": [...], "write": [...]}` block, and kept the schemas in `App/Karl/setup/model/`. The current generator no longer reads that format. Convert those files to the per-role `crud` object shown above.
