---
title : "jq"
description: "Reference documentation for the jq middleware"
lead: "The jq middleware executes json queries against inputs and outputs."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "middlewares"
weight: 300
---

This middleware is implemented with [gojq](https://github.com/itchyny/gojq) and
so inherits the same caveats and differences from the official jq project.

## fnrunner Key
`fnrun.middleware/jq`

## Configuration
The jq middleware must be configured by an object with an `input` key, `output`
key, or both. The value of either key must be a valid jq command and will be
applied to the input or output.

The jq middleware can be applied to map values such as those from the http 
source.

## Examples
The following example selects the value associated with the `x` key from the 
json body on an incoming request and passes it to the function.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
  - fnrun.middleware/json:
      input: deserialize
  - fnrun.middleware/jq:
      input: .x
fn: fnrun.fn/identity
```