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

## Config 

You can Set you Config for host and database and some config.

Also we can set what you want to use in **front-end** and what you want to use in **back-end**.

## Start New Project

Use Composer to start you project

```bash
composer create-project puneetxp/the_template_php project-name
```
## Run Setup

We have to Run setup for install what we need just bash/terminal code only.

```bash
npm init @angular angular
cd angular
cd ..
php setup.php
cd php
composer install puneetxp/the
```

Then We have to setup data you can run [setup →](/docs/model/setup)

##   Schema
We have this structure which are models
```bash
..
├── database/
│   └── Model/
|        ├── user.json
|        ├── brand.json
|        ├── role.json
|        └── active_role.json
```
Not Only Database Design We actually do more then that we Set Up crud (more then crud) you have to check [Schema →](/docs/model/schema)

## Other commands

Doks comes with commands for common tasks. [Commands →]({{< relref "commands" >}})
