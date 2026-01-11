---
title: "Golang"
description: "Documentation for Golang Generator"
lead: ""
date: 2026-01-11T16:24:00+05:30
lastmod: 2026-01-11T16:24:00+05:30
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "golang-gen"
weight: 1020
toc: true
---

## Overview

The `Golang` generator creates Go structs and Gin-Gonic compatible controllers.

## Configuration

Add `"golang"` to your `back-end` in `config.json`:

```json
{
  "back-end": ["golang"]
}
```

## Generated Structure

The Generator will output the following in the `go/` directory:

- `models/`: Contains Go structs with JSON tags.
- `controllers/`: Contains Controller functions for handling HTTP requests.

### Model Example

```go
package models

import (
    "time"
)

type User struct {
    Id int64 `json:"id"`
    Name string `json:"name"`
    // ... other fields
}
```

### Controller Example

```go
package controllers

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

type UserController struct{}

func (UserController) Index(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"data": "List of User"})
}
```
