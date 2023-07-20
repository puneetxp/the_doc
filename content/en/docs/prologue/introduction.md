---
title: "Introduction"
description: "The Framework kind of that is have same structured in all language. PHP , Deno, Go and Rust. Also More can possible."
lead: "The Framework have same structured in all language. PHP , Deno, Go and Rust. Also More can possible. Provide Front end binding for services and State. With some funtions over the top."
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

## Get started
You can get started with composer. 
```bash
composer project puneetxp/the_setup
```

For Starter you can use model in _setup/model and run the a php script on _setup directory name **autosetup.php** you can run 
```bash
php _setup/autosetup.php
```

### Tutorial

{{< alert icon="👉" text="The Tutorial is intended for novice to intermediate users." />}}

We can create json files in Model Folder, Like Name Role which is default Model Folder.

```json
{
    "name": "user",//MODEL NAME
    "table": "users",//TABLE NAME
    "crud": ["c", "r", "u","d"],//CREATE UPDATE READ DELETE
    "data": [{
            "name": "password",//name of database
            "mysql_data": "varchar(255)",//mysql database
            "sql_attribute": "NULL",//mysql sql_attribute 
            //if there is no sql_attribute then it is NOT NULL so you can skip it
            "datatype": "string"//for interface file
        }],
    "roles": {
        "read": ["admin"],//roles allow to read
        "update": ["admin"],//roles allow to update
        "write": ["admin"],//roles allow to write
        "delete": ["-"]//roles allow to delete
    }
}
```

```bash
..
├── _setup/
│   └── model/
|        ├── user.json
|        ├── role.json
|        └── active_role.json
├── php/
|    ├── App/
|    ├── The/
|    └── TheDep/ 
|    #TheDep is some dependency probably going to transfer to the based lib on composer / node / more
└── any_front_end/
|    ├── dist/
|    |   ├── .htaacess 
|    |   ├── index.php
|    |   └── api.php
|    └── ...source
```

### Quick Start

{{< alert icon="👉" text="The Quick Start is intended for intermediate to advanced users." />}}

One page summary of how to start a new Doks project. [Quick Start →]({{< relref "quick-start" >}})

## Go further

Recipes, Reference Guides, Extensions, and Showcase.

### Recipes

Get instructions on how to accomplish common tasks with Doks. [Recipes →](https://getdoks.org/docs/recipes/project-configuration/)

### Reference Guides

Learn how to customize Doks to fully make it your own. [Reference Guides →](https://getdoks.org/docs/reference-guides/security/)

### Extensions

Get instructions on how to add even more to Doks. [Extensions →](https://getdoks.org/docs/extensions/breadcrumb-navigation/)

### Showcase

See what others have build with Doks. [Showcase →](https://getdoks.org/showcase/electric-blocks/)

## Contributing

Find out how to contribute to Doks. [Contributing →](https://getdoks.org/docs/contributing/how-to-contribute/)

## Help

Get help on Doks. [Help →]({{< relref "how-to-update" >}})
