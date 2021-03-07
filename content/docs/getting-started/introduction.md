---
title : "Introduction"
description: "Introduction to fnrun and its concepts"
lead: "The fnrun project provides a set of tools for running business functions as a service."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "getting-started"
toc: true
weight: 100
---

The goal of fnrun is to provide a simple and extensible way to run business
functions as a service. Its approach separates infrastructure concerns from the
business functions, enabling application developers to build small 
business-focused functions and test in a variety of environments.

The fnrun project consists of an application called fnrunner and a Go library 
for building custom business functions and function runners.

## Concepts
fnrun contains four main concepts: sources, middleware, fns, and runners.

A source is a component that provides inputs to a business function and will 
handle the outputs. Some common sources include a web server that will receive 
HTTP requests and return HTTP responses and a queue client that will read 
messages from a queue and mark the messages as handled only if the business 
function returns an output successfully.

Middleware are components that process inputs received from sources before the 
input is sent to a business function. They also have an opportunity to 
manipulate the output or errors from the business function before being returned
to the source. Middleware can be composed into a middleware pipeline, where data
is passed through each middleware in order until the end of the pipeline is 
reached.

Fns are components that represent business functions. They can actually _be_ 
business functions written in Go, but they are more commonly components that 
interact with an external business function. As an example, the CLI fn runs a 
business function as a CLI application and communicates with it over std 
streams.

A runner is a combination of a source, middleware, and an fn. The fnrun project 
provides a runner called fnrunner that provides common sources, middleware, and 
fns. However, it is also possible to create custom runners that include new 
components designed to meet the specific needs of your environment.

## fnrunner
The fnrun project provides a runner called fnrunner which packages commonly used
sources, middleware, and fns. fnrunner is an executable program that accepts a 
YAML file (fnrun.yaml by default) that configures a processing pipeline for 
events.

Following is an example fnrun.yaml file that is used to build a process that 
runs an HTTP server on port 8080, selects the body from a request, executes the 
request from a pool of CLI applications run as `./myprogram`, and return the 
result as an HTTP response.

```yaml
source: fnrun.source/http
middleware:
  - fnrun.source/key: body
fn:
  fnrun.fn/pool:
    concurrency: 8
    template:
      fnrun.fn/cli: ./myprogram
```

Details about the use and configuration of the all available components in 
fnrunner are located on this site.

fnrunner also accepts overrides for the default fnrun.yaml. The following order
defines priority from highest to lowest:

* `CONFIG_FILE` environment variable
* `-f FILE` argument to fnrunner
* default `fnrun.yaml` file

## Deployment
The fnrun project is designed to be deployed in a wide variety of environments 
ranging from servers to serverless. Deploying an fnrun project requires a 
runner, a business function, configuration files such as fnrun.yaml, and any 
execution environment required by the business function itself. These may be 
packaged and deployed to an environment such as a Docker container by whatever 
means supported by your organization.

It should be noted that fnrun does not provide autoscaling capabilities, but
it is designed to work with systems that do provide this scalability such as
Knative, AWS Lambda, and AWS Fargate.
