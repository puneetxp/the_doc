---
title: "Java Spring"
description: "The the_spring helper library and the status of Spring generation."
lead: "`the_spring` is an early set of Java helpers. Spring generation is planned but not usable yet."
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-09-28T10:00:00+05:30
draft: false
images: []
weight: 1050
toc: true
---

{{< alert icon="⚠️" text="<strong>Status: skeleton.</strong> Putting <code>\"spring\"</code> in <code>back-end</code> currently does nothing. The generator is not wired into <code>setup.php</code>, and its templates are missing." />}}

## the_spring

The package is `com.puneetxp.lib`, under `src/main/java/com/puneetxp/lib/`:

| Class | Contents |
|---|---|
| `Model` | An abstract base with a nested `DB` interface (`rawSql`, `setPlaceholders`, `many`) and `join(...)`. |
| `SqlBuilder` | Builds LEFT JOIN queries. |
| `Auth` | `hash` (SHA-256). `login` and `register` are placeholders. |
| `Session` | A static map. In a real app, use `HttpSession` instead. |
| `Response` | Wraps output as `{ "data": … }`. |
| `FileAct` | Saves Spring `MultipartFile` uploads. |
| `Mail` | A stub. |

There is no `pom.xml` or `build.gradle` yet, and `DB` has no implementation.

## What generation would produce

The dormant generator (`compile-php/src/Class/javaspringset.php`) writes to `spring/src/main/java/com/example/demo/`:

- `model/<Name>.java`: a JPA `@Entity` with Lombok `@Data`, and `@Id @GeneratedValue` on `id`;
- `repository/<Name>Repository.java`: `extends JpaRepository<Name, Long>`;
- `controller/<Name>Controller.java`: `@RestController @RequestMapping("/api/<name>")` with repository-backed CRUD.

The entities import `javax.persistence`. Spring Boot 3 requires `jakarta.persistence`, so the generator needs that update before its output will compile.
