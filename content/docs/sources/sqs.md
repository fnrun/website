---
title : "sqs"
description: "Reference documentation for the sqs source"
lead: "The http source receives inputs from AWS SQS."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "sources"
weight: 400
---

The sqs source polls an AWS SQS queue for messages and transforms them into 
inputs for fns. If the fn returns a successful value, the source will mark the
message as handled. Otherwise, the message will become available for reading
again after the end of the visibility timeout, subject to the redrive policy set
on the queue.

## fnrunner Key
`fnrun.source/sqs`

## Configuration
The following table describes the keys on the configuration object for the sqs
source.

| Key             | Type    | Default       | Description                                                   |
|-----------------|---------|---------------|---------------------------------------------------------------|
| `queue`         | String  | `""`          | The name of the queue to poll for messages                    |
| `timeout`       | Integer | `30`          | The visiblity timeout for a message in seconds                |
| `batchSize`     | Integer | `1`           | The maximum number of messages to receive in a batch          |
| `endpointURL`   | String  | `""`          | The endpoint URL for SQS (default uses official AWS endpoint) |
| `partitionID`   | String  | `"aws"`       | The SQS partition ID                                          |
| `signingRegion` | String  | `"us-east-1"` | The SQS signing region                                        |

## Inputs
Inputs generated by the sqs source are objects with the following keys.

| Key    | Type   | Description      |
|--------|--------|------------------|
| `id`   | String | The message ID   |
| `body` | String | The message body |

## Examples
The following configuration defines a system that polls a queue named `my-queue`
for messages and posts each message to an external http endpoint. If the 
endpoint returns with a successful value (a 200- or 300-level status code), then
the message will be deleted from the system. Otherwise, the message will become
available again after the visibility timeout expires.

```yaml
source:
  fnrun.source/sqs:
    queue: my-test-queue
middleware:
  - fnrun.middleware/key: body
fn:
  fnrun.fn/http:
    targetURL: http://example.com/some-endpoint
```

The sqs source also exposes resolver configuration. This is particularly useful
for testing locally (e.g., with [LocalStack](https://localstack.cloud/)). The 
following example shows a configuration that will work with LocalStack when the
`ENDPOINT_URL` env var is set to `"http://localhost:4566"` and will work with 
SQS when the env var is not set.

```yaml
source: 
  fnrun.source/sqs:
    queue: my-test-queue
    endpointURL: $ENDPOINT_URL
middleware:
  - fnrun.middleware/debug
fn: fnrun.fn/identity
```