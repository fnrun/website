---
title : "circuitbreaker"
description: "Reference documentation for the circuitbreaker middleware"
lead: "The circuitbreaker middleware prevents fn invocations in the face of errors."
date: 2021-04-14T01:59:40+00:00
lastmod: 2021-04-14T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "middlewares"
weight: 300
---

## fnrunner Key
`fnrun.middleware/circuitbreaker`

## Configuration
The following table describes the keys on the configuration object for the 
circuitbreaker middleware.

| Key           | Type     | Default | Description                                                                       |
|---------------|----------|---------|-----------------------------------------------------------------------------------|
| `name`        | String   | `""`    | The name of the circuit breaker                                                   |
| `maxRequests` | Integer  | `1`     | The max number of requests to allow through when the circuit breaker is half-open |
| `interval`    | Duration | `60s`   | The period at which the internal counts are cleared                               |
| `timeout`     | Duration | `60s`   | The duration of the open state before moving into the half-open state             |

## Examples
The following applies a circuit breaker to prevent calling the remote endpoint
if it has had several failures in a small window of time.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.middleware/circuitbreaker:
      name: myBreaker
fn: 
  fnrun.fn/http:
    targetURL: http://example.com/some-endpoint
```