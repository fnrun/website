---
title : "Building and running business functions"
description: "Learn how to build and run business functions with fnrun"
lead: ""
date: 2021-03-15T18:45:00+00:00
lastmod: 2021-03-15T18:45:00+00:00
draft: false
images: []
menu: 
  tutorial:
    parent: "tutorial"
toc: true
weight: 111
---

In this part of the tutorial, we will build a business function for calculating momentum and build a JSON API around it.


## Write the business function
Let's start with the program Kelsey built for his [2018 KubeCon keynote](https://www.youtube.com/watch?v=oNa3xK2GFKY). For fun we should assume that [Dorothy Vaughn](https://en.wikipedia.org/wiki/Dorothy_Vaughan) really wrote something like this while she was learning and teaching Fortran. The following is the source code in its entirety (used with permission). Create a new directory and copy it into a file called `momentum.f90`.

```fortran
program function
        implicit none
        real :: mass, velocity, momentum

10      format(f0.3,',',f0.3,',',f0.3)

        do
                read(*,*,end=30)mass,velocity
                call calculate_momentum(mass,velocity,momentum)
                write(*,10)mass,velocity,momentum
        end do
30      continue

end program function

subroutine calculate_momentum(mass,velocity,momentum)
        implicit none
        real :: mass, velocity, momentum
        momentum = mass * velocity
end subroutine
```

This program will serve as our business function. The important thing to note is the contract of the program: it reads inputs over stdin in the form `mass,velocity` and prints the corresponding outputs to stdout in the form `mass,velocity,momentum`. The program is simple, but it is easy to imagine a much more complicated one developed by a subject matter expert.

We will worry about building the program later when we create a Dockerfile. We will not change this code at any point for the rest of the tutorial. All other changes we make will be to provide application infrastructure.

## fnrun components
fnrun has four major concepts: source, middleware, fn, and runner. A source is a software component that sits at the edge of your system, receiving inputs and generally interfacing with some external entity or service. An HTTP server is a good example of a source. It accepts requests from over the network and provides them as inputs to a business function. It also accepts the output from the function and uses them as responses. However, the request itself may not be suited for input directly into the business function, and similarly the output from the function may not be suitable for the source handle.

Middlewares solve this problem. A middleware is a piece of software that accepts input from a source, passes it along to a function, and receives and returns the result. However, middleware also get an opportunity to manipulate the input and output before passing it on to the next component. Middlewares can also be composed together into a pipeline. As an example, a middleware might take an input that is an HTTP request, select the body, and pass it on to the next component.

Finally, fns are components that interface with business logic. For example, the fn may create an instance of some external program, write the input to the program's stdin stream, and read a result from the program's stdout stream. It is then the responsibility of that external program to actually execute the business logic.

The source, middlewares, and fn form a processing chain. Inputs enter the system via the source, pass through the middleware chain, and finally get handled by the fn. The result from the fn is passed backwards through the middleware chain and is finally received by the source so it can complete an interaction with an external entity.

A runner is a software component that composes sources, middlewares, and fns into these processing chains. You will usually provide configuration information to a runner so that it knows how to compose a source, a list of middleware, and an fn into a fully functioning system. fnrun ships a runner called fnrunner. We will use fnrunner in the next section. However, fnrunner only houses the components supported by fnrun. Some use cases may dictate new components to satisfy a specific business need. In Part 3 of the tutorial, we will explore how to create new components and build custom runners.

## Define the JSON API
The first thing that we would like to do is expose the business function as a JSON API. It should accept an object with properties `mass` and `velocity` as input and return an object with the resulting `momentum`.

Let's think about what is required. We know contract of the function, the structure of its input and output. We also know that we need an HTTP server. However, there has to be some way to wire these two pieces together. We will need middleware for that. Finally we need an fn that can call the Fortran we defined in a previous step.

We will use fnrunner and build a configuration file to tell it how to build the application infrastructure we need.

Create a file called `fnrun.yaml` and fill out the skeleton as follows.

```yaml
source:
middleware:
fn:
``` 

Let's take the application and create a web API. We know that we want our source to be an HTTP server. Ideally, we would send a JSON request and get a JSON response. Luckily, fnrun provides a source that is an HTTP server. We know that the output from our business function will be text and not an HTTP response, so let's configure the http source to treat output as the body of a response. We also want the server to return a JSON response, so let's also set the value of the Content-Type header. Replace the `source` entry in the configuration file with the following.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
    outputHeaders:
      content-type: application/json
```

The key `fnrun.source/http` tells fnrunner that you want your source to be an http server, and you provide additional configuration information. The [doc page](https://fnrun.dev/docs/sources/http/) tells the name of the key to use to tell fnrunner you want to use this source as well as providing information about all the configuration options and defaults. For example, we know the server described above will connect on port 8080 and will accept HTTP but not HTTPS connections.

Next, we should define the function. fnrun provides a [cli fn](https://fnrun.dev/docs/fns/cli/) that runs an external program. Since our business function will continually read from stdin and write to stdout, it is considered a long-running program. The cli fn will monitor the program and restart it automatically if it crashes or exits unexpectedly.

All the cli fn definition requires is the command. In the next section, we will build the Fortran code into a program called `momentum`. Replace the fn definition in `fnrun.yaml` with the following.

```yaml
fn:
  fnrun.fn/cli: ./momentum
```

All that is left to define the middleware that will process the input and output. We know from the http source doc page that it creates an input object that has many keys. In this case, we are only interested in the value of the request body, which is under the `body` key. The [key middleware](https://fnrun.dev/docs/middlewares/key/) selects the value from an input object, so we can use it to get the body.

After the key middleware, the input will be a string containing a JSON object. We need to convert the JSON object into a comma-separated value. The [jq middleware](https://fnrun.dev/docs/middlewares/jq/) is a good candidate to do this. It can execute jq commands to transform JSON objects into other values. However, there is a problem. The input value is a JSON string, but the jq middleware expects an object as input. We can address this mismatch by using a step between them to deserialize the JSON string into an object. The [json middleware](https://fnrun.dev/docs/middlewares/json/) provides this functionality. 

> Since middleware can have specific expectations about inputs and outputs, it is important to consult their documentation when you are building a pipeline to make sure that the data output from one middleware is valid input for the next.

Once the JSON is converted into an object, we can write a jq query to pull out the `mass` and `velocity` values and join them into a comma-separated string. Here is the middleware so far:

```yaml
middleware:
  - fnrun.middleware/key: body
  - fnrun.middleware/json:
      input: deserialize
  - fnrun.middleware/jq:
      input: '[.mass, .velocity] | join(",")'
```

This handles the input into the function. Now we need to consider output coming back out of the function. Inputs flow from the top of the middleware stack to the bottom, and outputs flow in the opposite direction.

We know that the output of the function will be a comma-separated value in the form `mass,velocity,momentum`, but we only care about the last value. We can use jq again to split the string, take the last value, convert it to a number, and put it into a JSON object. We then need to serialize the JSON object back into a string, and we can use the json middleware to do that. After that point, we do not need to make any additional changes. When the output reaches the source, it will take the serialized JSON string and use it as the body of the HTTP response.

> Helpful hint: The [debug middleware](https://fnrun.dev/docs/middlewares/debug/) prints inputs, outputs, and errors to the runner's stdout. It is handy to drop into a pipeline to verify data is transformed the way you expect.

After incorporating these changes, the final version of `fnrun.yaml` should look like the following listing. If you read through it line by line, you should be able determine exactly what the system is going to do.

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
  - fnrun.middleware/jq:
      input: '[.mass, .velocity] | join(",")'
      output: '{ "momentum": split(",") | .[2] | tonumber }'
fn:
  fnrun.fn/cli: ./momentum
```

All that is left is to build our business function, download fnrunner, and package everything together so we can test it.


## Integrate the pieces into a system
We are almost finished. We have written our business function and defined the application architecture we want, and determined that the fnrunner meets our needs without customization. It is time to pull everything together. We will do that in a Docker container for convenience.

Create a file called `Dockerfile` and copy the following contents into it. We will walk through it step by step.


```docker
FROM alpine:latest AS momentum-builder

RUN apk add g++ gfortran
COPY . .
RUN gfortran -static -o momentum ./momentum.f90

## -----------------------------------------------------------------------------

FROM alpine:latest AS fnrunner-fetcher

ENV FNRUNNER_VERSION=0.4.0
RUN wget "https://github.com/fnrun/fnrun/releases/download/v${FNRUNNER_VERSION}/fnrunner_${FNRUNNER_VERSION}_linux_amd64.tar.gz" \
  && tar zxf ./fnrunner_${FNRUNNER_VERSION}_linux_amd64.tar.gz

## -----------------------------------------------------------------------------

FROM scratch

EXPOSE 8080

COPY --from=momentum-builder /momentum .
COPY --from=fnrunner-fetcher /fnrunner .
COPY ./fnrun.yaml .

CMD ["/fnrunner"]
```

This is a multi-stage Dockerfile. I like to separate stages with comment lines (e.g., `## -----`) to help improve readability. There are three stages in this file. In the first stage, we install a Fortran compiler, copy in our code and compile an executable called `momentum`. In the second stage, we download fnrunner from the fnrun GitHub project and mark the file as an executable. The third and final stage is the image that we will run.

The third stage starts with an empty Docker image. It copies in the three files necessary to run the application: the `momentum` program, the fnrunner, and the `fnrun.yaml` configuration file. Since we are running an HTTP server on port 8080, the image also exposes that port. Finally, we define the default command to start fnrunner. When fnrunner starts, it will load the configuration file and build the system we defined. After it has started, we can interact with the system.

Build the Docker image with the command `docker build -t fnrun-tutorial:0.1.0 .`, and then run the image with the command `docker run -p 8080:8080 fnrun-tutorial:0.1.0`. This will run the HTTP server and expose it on port 8080 on your localhost. In a new terminal window, you can interact with the server using curl. Here is a sample session:

```bash
➜  ~ curl -i -H 'Content-Type: application/json' -d '{"mass": 1.23, "velocity": 4.56}' localhost:8080
HTTP/1.1 200 OK
Content-Type: application/json
Date: Thu, 11 Mar 2021 05:36:13 GMT
Content-Length: 18

{"momentum":5.609}%
```

## Conclusion
At this point, you have learned the basics of using fnrun. In this tutorial, you learned about the basic components of fnrun, wrote a business function, defined the application infrastructure, and packaged it all together in a Docker image. In Part 2, we will deploy our application to Kubernetes and AWS Lambda.
