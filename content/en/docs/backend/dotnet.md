---
title: ".NET"
description: "The.DotNet.Lib runtime and the status of .NET generation."
lead: "`the_dotnet` is a .NET 10 port of the runtime (`The.DotNet.Lib`). The generator cannot produce .NET code yet."
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1030
toc: true
---

{{< alert icon="⚠️" text="<strong>Generation is not wired in.</strong> Putting <code>\"dotnet\"</code> in <code>back-end</code> currently does nothing: <code>setup.php</code> never calls the .NET generator, and its template folder is missing. Use the library by hand as shown below." />}}

## The.DotNet.Lib

The library targets `net10.0` and depends on ASP.NET Core, Identity, EF Core (Sqlite) and JWT Bearer. It is not published on NuGet yet, so reference the project directly:

```bash
dotnet add reference ../the_dotnet/Lib/The.DotNet.Lib.csproj
```

### DB

`DB` wraps any ADO.NET `DbConnection`. Each `?` placeholder is rewritten to `@p0`, `@p1`, … and bound as a parameter:

```csharp
using The.DotNet.Lib;
using Microsoft.Data.Sqlite;

IDB db = new DB(new SqliteConnection("Data Source=app.db"));
```

### Model

```csharp
public class Product : Model
{
    public Product(IDB db) : base(db) { Table = "products"; Name = "product"; }
}

var products = new Product(db);
var one    = products.Find(1);
var all    = products.All().Items;
var active = products.Where(new Dictionary<string, object> { { "enable", 1 } }).Items;
var page   = products.Paginate(1, 25);
```

The model also provides `Wherec`, `Count`, `Create`, `Update`, `Delete`, `Join`, `With(...)` and `Sort()`. `With` and `Sort` do eager loading the same way the PHP runtime does.

{{< alert icon="⚠️" text="Every statement runs through <code>ExecuteReader</code>, so <code>Update</code> and <code>Delete</code> return 0 instead of the number of rows affected. Table and column names are interpolated into the SQL, so never pass user input as a column name." />}}

### Auth

Auth is built on ASP.NET Identity and issues a JWT:

```csharp
await Auth.Register(userManager, email, password);
var result = await Auth.Login(userManager, signInManager, email, password);   // { token, email }
```

{{< alert icon="⚠️" text="The JWT signing key is hard-coded in the library. Replace it with a configured secret before any real deployment." />}}

The library also includes `Session`, which uses `AsyncLocal` to keep per-request values, plus `Response`, `FileAct` and `Mail` helpers.

## Sample API: `the_dotnet_api`

`the_dotnet_api` is an ASP.NET Core Web API that references the library. It exposes `POST api/auth/register` and `POST api/auth/login` using `{Email, Password}`, stores its data in Sqlite at `users.db`, and serves OpenAPI in Development.

```bash
cd the_dotnet_api && dotnet run     # http://localhost:5034
```

## What generation would produce

The dormant generator (`compile-php/src/Class/dotnetset.php`) writes `dotnet/Models/<Name>.cs` and `dotnet/Controllers/<Name>Controller.cs`. The controllers use `[Route("api/[controller]")]`, but their method bodies are stubs that return `Ok()`. The generated code does not use The.DotNet.Lib, and it ignores `crud` roles.
