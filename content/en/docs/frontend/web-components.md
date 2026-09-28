---
title: "Web Components"
description: "Framework-free custom elements from the_web_component."
lead: "`the_web_component` is a small set of vanilla custom elements that work with any frontend, or with none."
date: 2026-09-28T10:00:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1040
toc: true
---

{{< alert icon="👉" text="<strong>Prototype.</strong> The package isn't published yet. Build it from the repository." />}}

## Build

```bash
cd the_web_component
npm install
npm run dev      # demo at index.html
npm run build    # vite build --base=./
```

## `<editable-list>`

Shows a list that users can add items to. Type an entry and press **Enter** to add it as a chip. The component is styled with Tailwind classes.

```html
<editable-list
  title="Tags"
  add-item-text="Add a tag"
  list-item-1="php"
  list-item-2="deno">
</editable-list>
```

| Attribute | Meaning |
|---|---|
| `title` | The heading. |
| `add-item-text` | The placeholder for the input. |
| `list-item-N` | An initial item. Any number is allowed. |

## `<wysiwyg-bs>`

A `contentEditable` editor with a toolbar for bold, italic, underline and text alignment. It uses Bootstrap 5 and Bootstrap Icons.

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
<wysiwyg-bs></wysiwyg-bs>
```

The elements don't emit change events or use Shadow DOM yet. Read their values from the DOM.
