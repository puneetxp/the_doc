---
title: "DotNet"
description: "Documentation for .NET Generator"
lead: ""
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-01-11T16:24:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "dotnet-gen"
weight: 1010
toc: true
---

## Overview

The `.NET` generator works by reading your JSON models and creating corresponding C# Models and Controllers.

## Configuration

Add `"dotnet"` to your `back-end` in `config.json`:

```json
{
  "back-end": ["dotnet"]
}
```

## Generated Structure

The Generator will output the following in the `dotnet/` directory:

- `Models/`: Contains C# classes representing your database tables.
- `Controllers/`: Contains `ApiController` classes with basic CRUD operations.

### Model Example

```csharp
using System;
using System.Collections.Generic;

namespace DotNetApp.Models
{
    public class User
    {
        public long Id { get; set; }
        public string Name { get; set; }
        // ... other properties
    }
}
```

### Controller Example

```csharp
using Microsoft.AspNetCore.Mvc;
// ...

namespace DotNetApp.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UserController : ControllerBase
    {
        // GET, POST, PUT, DELETE methods
    }
}
```
