---
title : "timeout"
description: "Reference documentation for the timeout middleware"
lead: "The timeout middleware terminates an fn execution if it does not complete within a specified period."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "middlewares"
weight: 300
---

## fnrunner Key
`fnrun.middleware/timeout`

## Configuration
The timeout middleware accepts an optional string (default: 30s) describing the 
duration as configuration. The string will be parsed by
[time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) internally, so
it can be configured with strings compatible (e.g., `500ms`, `2h45m`).

## Examples
The following example specifies a 500-millisecond timeout on any fn execution.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.middleware/timeout: 500ms
fn: 
  fnrun.fn/cli: ./my-potentially-long-running-fn
```