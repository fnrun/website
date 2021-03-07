---
title : "json"
description: "Reference documentation for the json middleware"
lead: "The json middleware serializes and deserializes json on inputs and outputs."
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
`fnrun.middleware/json`

## Configuration
The json middleware accepts an object with keys `input` and `output`. Each key
may be set to either `serialize` or `deserialize`. If a key is not present,
the json middleware does not perform any operations on the value.

## Examples
The following example demonstrates a system that deserializes a json string
input and serializes a structure back to json.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
    outputHeaders:
      content-type: application/json
middleware:
  - fnrun.middleware/key: body
  - fnrun.middleware/json:
      input: deserialize
      output: serialize
fn: fnrun.fn/identity
```

The following example shows how to configure a system in which the json
middleware will serialize an output value but will not affect the input value.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
    outputHeaders:
      content-type: application/json
middleware:
  - fnrun.middleware/key: body
  - fnrun.middleware/json:
      output: serialize
fn: fnrun.fn/cli ./myfunction
```