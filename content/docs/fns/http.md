---
title : "http " # the space here is unfortunately important to prevent conflicts with the http title under sources
description: "Reference documentation for the http fn"
lead: "The http fn posts inputs as requests to an http endpoint."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "fns"
weight: 200
---

The http fn is useful for creating event triggers for other functions, 
especially ones used with an autoscaling technology. If the http endpoint 
returns a 200- or 300-level response, the fn treats it as a successful call and
returns the body of the repsonse as output. Otherwise, it treats the call as a 
failure and returns the body of the response in an error.

## fnrunner Key
`fnrun.fn/http`

## Configuration

| Key           | Type   | Default            | Description                              |
|---------------|--------|--------------------|------------------------------------------|
| `targetURL`   | String | `""`               | The url to which requests will be posted |
| `contentType` | String | `application/json` | The content-type sent with the request   |

## Examples
The following example creates a system that receives messages from AWS SQS and
posts them to another http endpoint.

```yaml
source:
  fnrun.source/sqs:
    queue: my-queue
middleware:
  - fnrun.middleware/key: body
fn:
  fnrun.fn/http:
    targetURL: http://example.com/some-endpoint
```