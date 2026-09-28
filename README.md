# THE Framework documentation

The source for [the-doc.netlify.app](https://the-doc.netlify.app/), the documentation site for the THE framework family: the `compile-php` generator, the PHP and Deno runtimes, the Angular and SolidJS frontends, and related projects.

The site is built with [Hugo](https://gohugo.io/) and the [Doks](https://getdoks.org/) theme.

## Run locally

```bash
npm install        # also downloads Hugo 0.107 extended into node_modules/.bin/hugo
npm run start      # http://localhost:1313
```

## Build

```bash
npm run build      # outputs to ./public
```

Before you open a pull request, check that the build has no `ERROR` or `WARN` lines.

## Where the content lives

```text
content/en/docs/
├── prologue/    introduction, projects, quick start, commands
├── model/       schema, config, generator (setup.php)
├── frontend/    angular, solidjs, vuejs, web components
├── backend/     php, deno, python, dotnet, golang, spring
├── examples/    INTAX billing app
└── help/        how to update, troubleshooting, FAQ
```

- The sidebar is generated from these folders.
- Sections are ordered by the `weight` in each `_index.md`, and pages by the `weight` in their front matter. You don't need `menu:` entries.
- Use `{{< relref "page" >}}` for internal links, so that broken links fail the build.
- Home page layout: `layouts/index.html`.
- Site metadata: `config/_default/params.toml`.
- Top menu: `config/_default/menus/menus.en.toml`.

## Writing guidelines

- Document only what the code does today. Mark unfinished features as **experimental** or **planned**.
- When the runtimes behave differently, show the difference. PHP and Deno, and Deno `0.0.2` and `0.1.x`, all differ in CRUD verbs.

## License

The content is Apache-2.0. The Doks theme is MIT, © Henk Verlinde.
