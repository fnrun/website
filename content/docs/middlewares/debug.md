---
title : "debug"
description: "Reference documentation for the debug middleware"
lead: "The debug middleware logs inputs, outputs, and errors."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "middlewares"
weight: 300
---

Each log statement is prefixed with `debugMiddleware` and is printed to the
standard output stream. This middleware is useful during development to view 
data at a particular point in a middleware pipeline.

## fnrunner Key
`fnrun.middleware/debug`

## Configuration
The debug middleware accepts an optional boolean value that indicates whether printing is enabled (true by default).


### Examples
In the following example, the debug middleware will log inputs, outputs, and errors.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.middleware/debug
fn: fnrun.fn/identity
```

In the following example, the debug middleware is disabled and will not print
any log statements.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.middleware/debug: false
fn: fnrun.fn/identity
```