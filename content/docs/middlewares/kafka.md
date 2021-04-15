---
title : "kafka "
description: "Reference documentation for the kafka middleware"
lead: "The kafka middleware sends messages to Kafka topics."
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
`fnrun.middleware/kafka`

## Configuration
The following table describes the keys on the configuration object for the kafka
middleware.

| Key            | Type   | Default | Description                                                          |
|----------------|--------|---------|----------------------------------------------------------------------|
| `brokers`      | String | `""`    | A comma-delimited list of broker addresses                           |
| `successTopic` | String | `""`    | An optional topic for successful fn output                           |
| `errorTopic`   | String | `""`    | An optional topic for errors returned from an fn                     |
| `certFile`     | String | `""`    | The path to a certificate file required for a TLS connection         |
| `keyFile`      | String | `""`    | The path to a key file required for a TLS connection                 |
| `caFile`       | String | `""`    | The path to a CA file required for a TLS connection                  |
| `verifySSL`    | Bool   | `false` | Indicates whether the system should verify the SSL certificate chain |

## Examples

The following reads messages from `test-topic` and sends outputs and errors to
`success-topic` and `error-topic`, respectively.

```yaml
source:
  fnrun.source/kafka:
    brokers: 127.0.0.1:9092
    topics: test-topic
    ignoreErrors: true
    group: testGroup
middleware:
  - fnrun.middleware/key: value
  - fnrun.middleware/kafka:
      brokers: 127.0.0.1:9092
      successTopic: success-topic
      errorTopic: error-topic
fn: 
  fnrun.fn/cli: python3 ./app.py
```