---
title : "identity"
description: "Reference documentation for the identity fn"
lead: "The identity fn returns an input value as output."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "fns"
weight: 200
---

The identity fn is useful in testing or building systems in which the middleware
performs all the necessary work to handle an invocation.

## fnrunner Key
`fnrun.fn/identity`

## Configuration
N/A

## Examples
The following configuration describes a system that echoes JSON http requests.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
    outputHeaders:
      content-type: application/json
middleware:
  - fnrun.middleware/key: body
fn: fnrun.fn/identity
```