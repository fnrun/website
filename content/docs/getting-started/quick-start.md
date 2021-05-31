---
title : "Quick start"
description: "A quick-start tutorial for fnrun."
lead: "Let's build your first fnrun application in 5 minutes."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "getting-started"
toc: true
weight: 102
---

This quick start is made to get you up and running with fnrun as quickly as
possible. 

## Write the function
We start by building a business function. We will build this one with Python,
but you can use whatever technology makes the most sense for you. This function
will read inputs from stdin and print outputs to stdout.

We will stick with the "hello world" tradition for this function. Copy the
following code into a file called `greeter.py`.

```python
import sys

for input in sys.stdin:
  print(f'Hello, {input.strip()}!', flush=True)
```

## Define the configuration
With the function written, we now turn our attention to defining how we want
to define our system. We have to provide a source of inputs, any middleware to
process data in a pipeline, and an fn to handle the input.

For this case, we will create an HTTP server, select its body, and provide the
result as input to our greeter program. We also want the output from the
function to be used as the body of the HTTP response.

The following configuration builds a system with the specifications described.
Copy it into a file called `fnrun.yaml`.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
fn:
  fnrun.fn/cli: python ./greeter.py
```

## Build a container image
Finally, we need to create a Docker image packaging the python program, the
configuration file, and the official fnrunner application. Copy the following
into a file called `Dockerfile`.

```shell
FROM python:buster

ENV FNRUNNER_VERSION=0.3.0
RUN wget "https://github.com/fnrun/fnrun/releases/download/v${FNRUNNER_VERSION}/fnrunner_${FNRUNNER_VERSION}_linux_amd64.tar.gz" \
  && tar zxf ./fnrunner_${FNRUNNER_VERSION}_linux_amd64.tar.gz

EXPOSE 8080
CMD ["./fnrunner"]

COPY . .
```

In a terminal, build the Docker image with the command 
`docker build -t quick-start:latest .`.

## Run the system and interact with it
After building the container image, run it with the command
`docker run -p 8080:8080 quick-start:latest`. This will launch the HTTP server
and expose it on your localhost on port 8080. Try the following curl command
and see similar output.

```shell
➜  ~ curl -i -d 'World' localhost:8080
HTTP/1.1 200 OK
Date: Mon, 08 Mar 2021 05:33:48 GMT
Content-Length: 13
Content-Type: text/plain; charset=utf-8

Hello, World!%
```

## Conclusion
Congratulations - you have built and run your first fnrun application! We wrote
a business function, specified configuration for our application, and executed
it inside a container. However, we have just scratched the surface of what you
can do with fnrun. There are several other sources, middlewares, and fns that
ship with fnrunner, and they are all documented on this site. You can even
create your own and build custom runners that meet your specific needs. If you
would like a more comprehensive guide, please check out the 
[tutorial](/tutorial/introduction).