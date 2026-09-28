---
title: "Angular"
description: "Generated Angular services, NGXS state and forms, plus the the-angular UI library."
lead: "With `\"angular\"` in `front-end`, the generator writes typed services, NGXS state and form validation for every model. The `the-angular` npm package supplies the UI that consumes them."
date: 2026-01-11T16:26:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1010
toc: true
---

## Configure

```json
{
  "front-end": ["angular"],
  "angular": { "outputPath": "../public_html/manager", "assets": ["src/storage"] }
}
```

Create the Angular app next to `setup.php` before you run the generator:

```bash
npm init @angular angular
php setup.php
cd angular && npm install the-angular @ngxs/store @angular/material
```

- `outputPath` is written into `angular/angular.json`.
- The asset `"src/storage"` is symlinked to `storage/public` so that uploads resolve during development.

## Generated code

Everything is generated under `angular/src/app/shared/`:

| Path | Contents |
|---|---|
| `Interface/Model/<Name>.ts` | An interface matching the table. |
| `Service/Model/<Name>.service.ts` | The API service, wired to NGXS and IndexedDB. |
| `Ngxs/State/<Name>.state.ts` | The state, with selectors. |
| `Ngxs/Action/<Name>.action.ts` | The actions `Set`, `Add`, `Edit`, `Upsert` and `Delete`. |
| `Form/Validation/<Name>.ts` | Create and update validator maps for the model. |
| `db/tables.ts`, `Service/run.service.ts` | The IndexedDB table list, and start-up code that hydrates the stores. |

### Services

```ts
constructor(private brandService: BrandService) {}

ngOnInit() {
  this.brandService.prefix("isuper").all();   // → GET /api/isuper/brand
}

brands$ = this.brandService.allState();        // Observable<Brand[]> from the store
```

| Method | HTTP call | Store action |
|---|---|---|
| `prefix(role)` | Sets `url` to `/api/<role>/<model>` (the default is `/api/<model>`). | none |
| `all()` | `GET url` the first time; after that `GET url?latest=<newest updated_at>`. | `Set`, or `Upsert` on later calls |
| `fresh()` | `GET url` | `Set` |
| `get(id)` | `GET url/id` | none (returns an Observable) |
| `create(v)` | `POST url` | `Add` |
| `update(id, v)` | `PATCH url/id` | `Edit` |
| `upsert(rows)` | `PUT url` with body `{ <table>: rows }` | `Upsert` |
| `del(id)` | `DELETE url/id` | `Delete` |
| `allState()`, `getState(id, key)`, `array()` | none (reads the store) | none |
| `toggle(id)` | Flips `enable`. Only generated when the model has `enable`. | `Edit` |

Stores are mirrored to IndexedDB, so the UI renders cached data immediately and `all()` only fetches the rows that changed.

{{< alert icon="👉" text="The generated services send <code>PATCH</code> for update and <code>PUT</code> for upsert. That matches the PHP runtime and Deno <code>@puneetxp/the@0.1.x</code>. The older Deno <code>the@0.0.2</code> expects <code>POST /:id</code> for update." />}}

## the-angular library

```bash
npm install the-angular
```

Version `0.0.13` requires Angular 21, `@angular/material` 21, `@ngxs/store` 21 and `rxjs` 7.8. Import everything from the package root:

```ts
import { FormDynamicComponent, TableMaterialComponent, setformbase, predefined, IndexedDBService } from "the-angular";
```

### Components

| Selector | Purpose |
|---|---|
| `<the-form-dynamic>` | Renders a form from a `FormBase[]` description. |
| `<the-input-dynamic>` | Renders a single dynamic field. |
| `<the-table-material>` | A filterable, paginated Material table with edit, delete and enable actions. |
| `<the-sidenav>`, `<the-footer>`, `<the-page-title>` | Layout. `the-sidenav` takes `[menus]`. The header component exists in the source but is not exported. |
| `<the-upload-image>` | Image upload with a preview. |
| `<the-login-dialog>` | A login dialog wired to `AuthService`. |
| `<the-skeleton>`, `<the-sort>` | A loading placeholder and a sort control. |
| Not-found and not-allowed pages | Route targets. |

### Tables

```html
<the-table-material
  [table_mat$]="brandService.allState()"
  [columnsToDisplay]="['name', 'slug', 'enable']"
  [isfilter]="true" [isPaginate]="true" [PageSize]="25"
  [isEdit]="true" [isDelete]="true" [isEnable]="true"
  (edit)="edit($event)" (delete)="brandService.del($event)" (enable)="brandService.toggle($event)">
</the-table-material>
```

### Forms

Describe the fields, attach the generated validation, and render:

```ts
// Form.ts
export const Form = [
  { key: "name",  label: "Name",  controlType: "textbox", row: "col-span-2" },
  { key: "type",  label: "Type",  controlType: "dropdown", options: [{ key: "a", value: "A" }] },
  predefined.slug,
  predefined.enable(1),
];

// add.component.ts
import { CreatebrandForm } from "src/app/shared/Form/Validation/Brand";
inputs = setformbase(Form, [CreatebrandForm, {}]);
```

```html
<the-form-dynamic [inputs]="inputs" [formClass]="'grid grid-cols-4 gap-2'" (formOutput)="brandService.create($event)" />
```

- **`controlType` values:** `textbox`, `textarea`, `password`, `hidden`, `dropdown`, `select`, `dropdownautocomplete`, `chipselect`, `checkbox`, `toggle`, `datepicker`, `date` and `photo`.
- **Validators** go in `validator: ValidatorFn[]` on each field. `setformbase` fills them in from the generated validation map.
- **`predefined`** provides ready-made fields: `seoform`, `_name`, `slug`, `phone`, `email`, `description`, `enable(v)` and others.

### Services and state

| Export | Purpose |
|---|---|
| `AuthService` | `POST /api/login`, `GET /api/logout` and Google sign-in. Dispatches `SetLogin`. |
| `LoginState`, `SetLogin`, `DeleteLogin` | The NGXS login state, with `getLogin` and `isLogin` selectors. |
| `IndexedDBService` | `The_putSomeData`, `The_getAllData` and `The_delSomeData`, used by the generated services. |
| `DialogService`, `DynamicFormService`, `FormDataService`, `ImageService` | UI and HTTP helpers. |
| `initservice(services, prefix)` | Calls `prefix(prefix).all()` on every service in the list. |

## Legacy: the-angular-material

`the-angular-material` is the Angular 16 predecessor of `the-angular`. It is no longer maintained, and its library build points to a missing `src/public-api.ts`. Use `the-angular` instead.
