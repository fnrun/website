---
title : "cron"
description: "Reference documentation for the cron source"
lead: "The cron source triggers a function on a schedule."
date: 2021-04-14T01:59:40+00:00
lastmod: 2021-04-14T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "sources"
weight: 400
---

## fnrunner Key
`fnrun.source/cron`

## Configuration
The configuration is a [cronspec](https://crontab.guru) that supports seconds
as an optional input as well as non-standard descriptors such as `@weekly`.

## Inputs
The input is an empty object.

## Examples
The following example shows how to invoke a function once every second.

```yaml
source:
  fnrun.source/cron: '@every 1s'
fn:
  fnrun.fn/cli: node ./function.js
```

The following example also invokes a function once every second. It demonstrates
the optional seconds entry.

```yaml
source:
  fnrun.source/cron: '* * * * * *'
fn:
  fnrun.fn/cli: node ./function.js
```
