---
title: "Config"
description: ""
lead: ""
date: 2023-08-09T16:57:25+05:30
lastmod: 2023-08-09T16:57:25+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "config-346d7db95c547a1f9c75dcf2afb05474"
weight: 1000
toc: true
---
We have config.json on root of porject
## Basic
Basic Setting for the Database and Session genration
```json
{    
  "env": {
    "host": "hostname.com", // hostname
    "sslhost": ".hostname.com", // hostname for ssl
    "dbhost": "localhost", // database host
    "dbname": "demo", // database name
    "dbuser": "username", // database user
    "dbpwd": "password", // database password
    "samesite": "Strict", // samesite
    "httponly": true, // session
    "secure": true // session
  },
}
```

Then What we want to backend and front we have right now limited option like

## Back End
```json
{
  "back-end":[
    "php", // A Php App
    "deno", // A Deno App **Unstable
    "dotnet", // A .NET App
    "golang", // A Golang App
    "spring" // A Java Spring App
  ]
}
```
## Front End
```json
{
  "front-end":[
    "angular", // api integration and state management ngxs
    "solidjs", // api integration with solid store. **not tested for long time.
    "vuejs" // not functional
  ]
}
```

### Angular
Specific angular configuration
```json
"angular": {
    "outputPath": "..\/public_html\/manager",
    "assets": []
}
```

### Solidjs

See [SolidJS Documentation](/docs/frontend/solidjs).

### Vuejs

See [VueJS Documentation](/docs/frontend/vuejs).