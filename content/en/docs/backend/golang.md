---
title: "Go"
description: "The the_go helper library and the status of Go generation."
lead: "`the_go` is an early set of Go helpers that mirror the other runtimes. Go generation is planned but not usable yet."
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1040
toc: true
---

{{< alert icon="⚠️" text="<strong>Status: skeleton.</strong> Putting <code>\"golang\"</code> in <code>back-end</code> currently does nothing. The generator is not wired into <code>setup.php</code>, and its templates are missing." />}}

## the_go

The library is organised as packages under `utils/`:

| Package | Contents |
|---|---|
| `sqlbuilder` | `BuildJoinQuery` builds LEFT JOINs with `?` placeholders. |
| `model` | `Model{Table, DB, Relations, Items}` with `Join()`. |
| `auth` | `Hash` (SHA-256). `Login` and `Register` are placeholders. |
| `session` | A global map. It is not safe to share across requests. |
| `response`, `file`, `mail` | Helpers. `mail.Send()` is a mock. |

The repository has no `go.mod` yet, so it cannot be fetched with `go get`, and the `DB` interface has no implementation. It is a starting point for a port, not a working runtime.

## What generation would produce

The dormant generator (`compile-php/src/Class/golangset.php`) writes:

- `go/models/<name>.go`: structs with `json` tags;
- `go/controllers/<name>_controller.go`: Gin handlers (`Index`, `Show`, `Store`, `Update`, `Delete`) that return placeholder JSON.

The generated code does not use `the_go`, and it ignores `crud` roles.
