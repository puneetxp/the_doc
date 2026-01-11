---
title: "SolidJS"
description: "Documentation for SolidJS Generator"
lead: ""
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-01-11T16:26:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "solid-gen"
weight: 1020
toc: true
---

## Overview

The `SolidJS` generator produces a lightweight, reactive frontend structure using SolidJS, focusing on fine-grained reactivity.

## Configuration

Add `"solidjs"` to your `front-end` in `config.json`:

```json
{
  "front-end": ["solidjs"]
}
```

## Generated Structure

The Generator will output the following in the `solidjs/src/shared/` directory:

- `Interface/Model/`: TypeScript Interfaces.
- `Service/Services.ts`: Centralized Service exports.
- `run.ts`: Initialization logic, handling IndexedDB and initial data fetching.

*(Note: Stores and individual Service files are currently referenced in code comments but primarily generated within the centralized `Services.ts` pattern in the current version)*

### Service Pattern

Each model gets a generic `ModelService` instance configured with its specific endpoint URL.

```typescript
export const UserService = (new ModelService<User>())
    .seTable("user")
    .seturl("api/user");
```

### IndexedDB & Init

The `run.ts` file handles initializing the application state, checking login status, and populating tables from IndexedDB or the API.
