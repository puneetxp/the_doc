---
title: "Angular"
description: "Documentation for Angular Generator"
lead: ""
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-01-11T16:26:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "angular-gen"
weight: 1010
toc: true
---

## Overview

The `Angular` generator creates a robust Single Page Application (SPA) structure integrated with NGXS for state management and Angular Forms for validation.

## Configuration

Add `"angular"` to your `front-end` in `config.json`:

```json
{
  "front-end": ["angular"],
  "angular": {
      "outputPath": "../public_html/manager",
      "assets": []
  }
}
```

## Generated Structure

The Generator will output the following in the `angular/src/app/shared/` directory:

- `Interface/Model/`: TypeScript Interfaces matching your DB schema.
- `Service/Model/`: Angular Services dealing with HTTP requests.
- `Ngxs/State/`: individual States for each model.
- `Ngxs/Action/`: Actions (Add, Edit, Delete, Set, Upsert) for each model.
- `Form/validation/`: Form validation configurations.

### Key Features
- **NGXS State Management**: Automatically creates state, actions, and selectors.
- **IndexedDB Support**: Includes client-side caching mechanisms via IndexedDB via `RunService`.
- **Form Generators**: creates validator configurations for Create and Update forms.

### Example Usage

The services generated are designed to work seamlessly with the Store.

```typescript
// Component Code
constructor(private store: Store, private userService: UserService) {}

ngOnInit() {
  this.userService.all(); // Fetches from API and updates Store
}
```
