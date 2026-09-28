---
title: "VueJS"
description: "Status of Vue generation."
lead: "Vue support is planned, but it doesn't generate anything yet."
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1030
toc: true
---

{{< alert icon="⚠️" text="<strong>Not functional.</strong> The generator recognises <code>\"vuets\"</code> in <code>front-end</code>, but its constructor never runs the <code>set()</code> step, so no files are written. It also prints <code>Angular Build</code> by mistake." />}}

## Configuration

```json
{ "front-end": ["vuets"] }
```

The value is `vuets`. `vuejs` is not recognised.

## Planned output

Once it is fixed, the generator (`compile-php/src/Class/vueset.php`) is designed to write the following under `vuets/src/shared/`:

| Path | Contents |
|---|---|
| `Interface/Model/<Name>.ts` | TypeScript interfaces |
| `Store/Model/<Name>` | Pinia stores (`defineStore`) holding `rawItems` and actions such as `addItem`, `removeItem` and `upsertItem` |
| `Service/Model/<Name>` | `fetch` wrappers that update the store after each call |

Until then, use the [Angular]({{< relref "angular" >}}) or [SolidJS]({{< relref "solidjs" >}}) generator. Another option is to consume the REST API directly, using the interfaces generated for another frontend.
