---
title: "Quick Start"
description: "One page summary of how to start a new The project."
lead: "One page summary of how to start a new The project."
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 110
toc: true
---

## Requirements

- [PHP 8.2](https://www.php.net/downloads.php/)
- [Composer](https://getcomposer.org/download/)
- [Nodejs](https://nodejs.org/en)

<!-- {{< details "Why Composer?" >}}
The uses composer to centralize dependency management.
{{< /details >}} -->

## Start New Project

Use Composer to start you project

```bash
composer create-project puneetxp/the_template_php project-name
```

##   Schema
We have this structure which are models
```bash
..
├── _setup/
│   └── model/
|        ├── user.json
|        ├── brand.json
|        ├── role.json
|        └── active_role.json
```
Not Only Database Design We actually do more then that we Set Up crud (more then crud)
### Example

setup/model/brand.json
{{< details "A Demo Model" >}}

```json
{
  "name": "brand", //MODEL NAME
  "table": "brands", //TABLE NAME
  "crud": { //DIFFERENT ROLES
    "isuper": ["c", "r", "u", "a", "p", "d"], 
    //CREATE(c) READ(r) UPDATE(u) ALLREAD(a) INSERTMUTIPLE(p) DELETE(d)
    "ipublic": ["a", "r" ],
    "roles":[//some roles
      "some_roles":["c","r"]
    ]
    },
    "enable": 1, //ENABLE TABLE
    "addtional": ["seo", "slug"], //SOME ADDTIONAL
  "data": [
    {
      "name": "name", //NAME OF COLUMN
      "mysql_data": "varchar(255)", //MYSQL DATATYPES
      "datatype": "string" //TYPESCRIPT DATATYPE
    }
  ],
  "relation": [ //
    "check"//RELATION TO MODEL NAME
    {
      "name": "photo", //RELATION TO MODEL NMAE
      "default": "NULL" //DEFAULT VALUE MYSQL
    }
  ]
}

```
{{< /details >}}
Then We have to setup data **shecma** you can run  [Schema →](/docs/model/schema)

## Config 

You can Set you Config for host and database and some config.

Also we can set what you want to use in **front-end** and what you want to use in **back-end**.

### Example


{{< details "A Demo Example" >}}
```json
{
  "env": {
      "host": "hostname.com",
      "sslhost": ".hostname.com",
      "dbhost": "localhost",
      "dbname": "demo",
      "dbuser": "username",
      "dbpwd": "password",
      "samesite": "Strict",
      "httponly": true,
      "secure": true
  },
  "front-end": [
      "angular", //angular
      "solidjs", //solidjs
      "vuejs", //vuejs
  ],
  "back-end": [
      "php", //php
      "deno", //deno
  ]
}
```
{{< /details >}}
## Start a new Doks project

Create a new site, change directories, install dependencies, and start development server.

### Create a new site

Doks is available as a child theme and a starter theme.

#### Child theme

- Intended for novice to intermediate users
- Intended for minor customizations
- [Easily update npm packages]({{< relref "how-to-update" >}}) — __including__ [Doks](https://www.npmjs.com/package/@hyas/doks)

```bash
git clone https://github.com/h-enk/doks-child-theme.git my-doks-site
```

#### Starter theme

- Intended for intermediate to advanced users
- Intended for major customizations
- [Easily update npm packages]({{< relref "how-to-update" >}})

```bash
git clone https://github.com/h-enk/doks.git my-doks-site
```

{{< details "Help me choose" >}}
Not sure which one is for you? Pick the child theme.
{{< /details >}}

### Change directories

```bash
cd my-doks-site
```

### Install dependencies

```bash
npm install
```

### Start development server

```bash
npm run start
```

Doks will start the Hugo development webserver accessible by default at `http://localhost:1313`. Saved changes will live reload in the browser.

## Other commands

Doks comes with commands for common tasks. [Commands →]({{< relref "commands" >}})
