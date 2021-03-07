---
title : "Use cases"
description: "Descriptions for fnrun use cases"
lead: "fnrun supports a number of use cases."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "getting-started"
toc: true
weight: 101
---

fnrun is primarily used to separate the infrastructure configuration from an the
implementation of a business function. This support developers developing in a 
serverless function style even when deploying to server-based environments.

The following use cases describe why you might be motivated to adopt fnrun into
your organization.

# Treat existing CLI applications and scripts as business functions
fnrun provides a cli fn implementation that calls external applications and 
scripts and treats them as business functions. Those functions can read inputs 
from stdin and write outputs to stdout. Programs that implement this standard 
interface can already be treated as business functions!

# Test functions in isolation
Testing serverless functions is notoriously difficult. However, fnrun helps
developers invoke functions from a number of environments.

Having a hard time running Kafka locally? Test with an HTTP server instead, 
providing a middleware pipeline to transform the data into the same form as 
would be received from Kafka. While this approach definitely is not the ideal,
it is certainly better than only being able to test a function by deploying it.

# Standardize infrastructure across an organization
fnrun is configurable at runtime and is open for extension. This means that your
organization can create new sources, middleware, and fns appropriate for your
specific needs.

Additionally, the fnrun configuration can be separated from the definitions of
business functions. This allows different processes to manage functions and the
infrastructure set up to support those functions. One team may manage the 
security configuration of the application while the business function developers
focus only on the business logic required to provide business value.

# Create hybrid deployment environments
Since fnrun supports both server-based and serverless environments using the
business functions, businesses can deploy into whatever environment makes the 
most sense. Long-running functions might be deployed as a service in a 
Kubernetes cluster while the majority of functions are deployed to a serverless
system.

You can even run functions on-prem to the degree the scale that your hardware
supports and then scale into the cloud, using the same function implementation
in both places.