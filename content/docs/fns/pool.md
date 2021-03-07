---
title : "pool"
description: "Reference documentation for the pool fn"
lead: "The pool fn distributes invocations among a collection of instances of an fn."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "fns"
weight: 200
---

The pool fn creates and executes many instances of another fn as defined by a
template configuration value. The pool ensures that each function handles an
input in a synchronous fashion, and it tracks which instances of the template fn
are available for handling at any point in time. If no instances are available,
it will wait for an instance to become available subject to a wait duration
timeout. The default wait duration is 500ms.

## fnrunner Key
`fnrun.fn/pool`

## Configuration

| Key               | Type             | Default | Description                                                                                                                        |
|-------------------|------------------|---------|------------------------------------------------------------------------------------------------------------------------------------|
| `concurrency`     | Integer          | `8`     | The number of instances of the template to create                                                                                  |
| `maxWaitDuration` | Duration         | `500ms` | The duration to wait for the availability of an fn instance. The invocation returns an error if the max wait duration is exceeded. |
| `template`        | Fn Configuration | `nil`   | A description of the fn to run in a pool                                                                                           |

## Examples
The following example demonstrates how to build a system that starts three
copies of an external program to handle inputs with a maximum wait duration of
one second.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
fn: 
  fnrun.cli/pool:
    concurrency: 3
    maxWaitDuration: 1s
    template:
      fnrun.fn/cli: node ./fn.js
```