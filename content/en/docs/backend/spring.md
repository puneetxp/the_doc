---
title: "Java Spring"
description: "Documentation for Java Spring Generator"
lead: ""
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-01-11T16:24:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "spring-gen"
weight: 1030
toc: true
---

## Overview

The `Java Spring` generator creates standard Spring Boot Entities, Repositories, and REST Controllers.

## Configuration

Add `"spring"` to your `back-end` in `config.json`:

```json
{
  "back-end": ["spring"]
}
```

## Generated Structure

The Generator will output the following in the `spring/src/main/java/com/example/demo/` directory:

- `model/`: Entity classes annotated with JPA annotations.
- `repository/`: Interfaces extending `JpaRepository`.
- `controller/`: REST Controllers.

### Entity Example

```java
package com.example.demo.model;

import javax.persistence.*;
import lombok.Data;

@Entity
@Data
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

### Controller Example

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;
// ...

@RestController
@RequestMapping("/api/user")
public class UserController {
    // Standard CRUD endpoints
}
```
