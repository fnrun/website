---
title : "ratelimiter"
description: "Reference documentation for the ratelimiter middleware"
lead: "The ratelimiter middleware restricts the frequency of calls to a rate with a maximum burst."
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
`fnrun.middleware/ratelimiter`

## Configuration
The following table describes the keys on the configuration object for the 
ratelimiter middleware.

| Key     | Type     | Default | Description                                       |
|---------|----------|---------|---------------------------------------------------|
| `burst` | Integer  | `1`     | The maximum number of requests allowed in a burst |
| `every` | Duration | `1s`    | The period at which a call token is created       |

## Examples
The following applies a ratelimiter to prevent calling the remote endpoint
more than twice per second.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.middleware/ratelimiter:
      every: 500ms
fn: 
  fnrun.fn/http:
    targetURL: http://example.com/some-endpoint
```