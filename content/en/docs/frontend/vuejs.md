---
title: "VueJS"
description: "Documentation for VueJS Generator"
lead: ""
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-01-11T16:26:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "vue-gen"
weight: 1030
toc: true
---

## Overview

The `VueJS` generator creates a modern Vue 3 application structure using Pinia for state management.

## Configuration

Add `"vuets"` to your `front-end` in `config.json`:

```json
{
  "front-end": ["vuets"]
}
```

## Generated Structure

The Generator will output the following in the `vuets/src/shared/` directory:

- `Interface/Model/`: TypeScript Interfaces.
- `Store/Model/`: Pinia Stores for each model.
- `Service/Model/`: API Service files for performing CRUD operations.

### Pinia Store Example

Stores are generated using the `defineStore` syntax:

```javascript
export const useUserStore = defineStore({
    id: "User",
    state: () => ({
        rawItems: [],
    }),
    actions: {
        addItem(User) { ... },
        removeItem(id) { ... },
        // ...
    }
})
```

### Service Pattern

Services encapsulate `fetch` calls and automatically dispatch actions to the corresponding Pinia store upon success.

```javascript
// Service/Model/User.js
import { useUserStore } from "/src/Store/Model/User.js";

function all() {
    fetch("/api/user")
      .then(r => r.json())
      .then(i => { useUserStore().upsertItem(i) });
}
```
