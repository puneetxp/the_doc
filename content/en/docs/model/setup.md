---
title: "Setup"
description: ""
lead: "Setup Script is install.sh/install.bat"
date: 2023-08-09T16:57:25+05:30
lastmod: 2023-08-09T16:57:25+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "schema-346d7db95c547a1f9c75dcf2afb05474"
weight: 998
toc: true
---

For Starting we can start with what config we need to install

This is Solidjs with DENO

## Solidjs with DENO
```bash
# sholidjs with deno
cd setup
php autosetup.php
cd ../
npx degit solidjs/templates/ts solidjs
cd solidjs
npm install
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm run dev
```
## Angular with php
We can do same with angular with php

```bash
php angular
npm init @angular angular
cd angular
cd ..
php setup.php
cd php 
composer install puneetxp/the
```

## Storage 
For Storage We Recommend to use 
```bash
ln -s ../storage/public ./public_html/storage
```
Possibilities is unlimited upto you terminal scripting 