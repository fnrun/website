---
title : "Extending fnrun"
description: "Learn how to write a new middleware and create a custom runner"
lead: ""
date: 2021-03-15T18:45:00+00:00
lastmod: 2021-03-15T18:45:00+00:00
draft: false
images: []
menu: 
  tutorial:
    parent: "tutorial"
toc: true
weight: 113
---

This is the final part of the tutorial. In this part, you will learn how to extend fnrun to meet your organization's unique needs.

## A new scenario
Let's pretend that we need to use our service in a new way. We want to pull messages from a queue that contain mass and velocity values and place a message on a different queue containing the momentum value. This scenario is of questionable value but will allow us to demonstrate a few new concepts.

One approach to doing this would be to use the SQS source to receive inputs and use the HTTP fn to call our existing deployment on Kubernetes. We could then use a middleware to place the result on a different queue.

Unfortunately, fnrun does not provide a middleware that publishes a message to a queue. In order to satisfy this business need, we will need to create a new middleware and package it into a custom runner for use within our organization.

## Creating a project
fnrun provides a [project template](https://github.com/fnrun/runner-template) for creating customer runners. We will use this template to create a custom runner and add our custom middleware to the project. On the GitHub page for the project template, press the 'Use this template' function and following the prompts to create a new project for us to work with.

Your first step should be to update the module name in `go.mod` with a name appropriate for your organization.

The two most important files in the project are `main.go` and `Makefile`. The main.go file contains the code to read a configuration file, bootstrap a runner, apply the configuration, and run the runner. The Makefile contains build commands for Linux, macOS, and Windows. The default action is to build all three.

## Writing middleware
Now that we have our project set up, we can start to build a middleware. fnrun provides interfaces for each of the major components. For our scenario, we will provide a new implementation of the Middleware interface in the github.com/fnrun/fnrun/run package.

> If you look at the structure of fnrun, you will notice that we split out fn and run top-level packages. The Fn interface is in the fn package, and the other interfaces are in the run package. 
>
> Why is the code split up this way? In addition to supporting building business function runners, fnrun also serves as an API for building business functions in Go. The fn package contains everything an application developer needs to building business functions. The run package is only focused on building runners.

### Implementing the middleware
Create a new file called `sqs.go`. We will use this file to contain our middleware. We should start by creating a struct to represent our middleware. This struct should contain everything we need to interface with SQS. Let's start with the following definition.

```go
type sqsMiddleware struct {
	svc      *sqs.SQS
	queueURL *string
}
```

The Middleware interface contains a single method `Invoke`. The Invoke method accepts a context, input, and fn. It returns an (output, error) pair. The input and output are both `interface{}` because fnrun cannot predict what types might be used between steps in a processing pipeline. In general, these values will be either primitives (e.g., integers, strings) or an object of type `map[string]interface{}`. 

The Invoke method will be invoked when it needs to process an input value. The middleware may manipulate the input value before calling the Fn, and it may manipulate the output from the Fn before returning. If an error is encountered, the middleware can return that error. Calling the Fn is optional. For example, if a middleware provides rate limiting, and the limit has been exceeded, the middleware can return an error without invoking the Fn.

The following code implements an Invoke method on our middleware. It accepts input and invokes the Fn. It then creates a message and sends it to SQS before returning the function output.

```go
func (sm *sqsMiddleware) Invoke(ctx context.Context, input interface{}, f fn.Fn) (interface{}, error) {
	output, err := f.Invoke(ctx, input)
	if err != nil {
		return output, err
	}

	body := fmt.Sprint(output)

	msg := sqs.SendMessageInput{
		MessageBody: &body,
		QueueUrl:    sm.queueURL,
	}
	_, err = sm.svc.SendMessage(&msg)
	if err != nil {
		return nil, err
	}

	return output, nil
}
```

### Registering the middleware
Now that we have defined our middleware, we need to register it with the runner. Open `main.go` and take a look at the code. You will see that the application starts creates a new registry and registers all the sources, middlewares, and fns provided with the official fnrunner.

We need to register a new middleware by calling `RegisterMiddleware` on the registry. The method accepts a key name that will be used in YAML configuration files and a factory function to create a new instance of the middleware. Let's go ahead and register our middleware by adding the following code.

```go
registry.RegisterMiddleware("my-organization.middleware/sqs", NewSqsMiddleware)
```

Now we need to provide the implementation of the `NewSqsMiddleware` in `sqs.go`. Add the following to the file.

```go
func NewSqsMiddleware() run.Middleware {
	return &sqsMiddleware{}
}
```

### Handling configuration data
If our middleware data did not need any additional information, we would be finished. However, we need additional information to configure our particular middleware. Specifically we need the name of the queue we will send messages to.

fnrun provides a config package with a number of interfaces such as `StringConfigurer` and `MapConfigurer`. These provide methods to receive configuration data in a specific type and apply to an object. In the default implementation, the configuration data is parsed from the input YAML file. In our case, we want to specify a string containing the queue name. It might appear like the following in a YAML file.

```yaml
my-organization.middleware/sqs: queue-name
```

It should be noted that your struct can implement many of the interfaces, and the config package will call the one based on the type of input data. If an appropriate implementation is not found, it will return an error. To support this scenario, we should implement the `StringConfigurer` interface. Copy the following code into `sqs.go`. The code will create an AWS session for SQS and set the values in our object so that it can be invoked.

```go
func (sm *sqsMiddleware) ConfigureString(queueName string) error {
	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	sm.svc = sqs.New(sess)

	urlResult, err := sm.svc.GetQueueUrl(&sqs.GetQueueUrlInput{
		QueueName: &queueName,
	})

	if err != nil {
		return err
	}

	sm.queueURL = urlResult.QueueUrl

	return nil
}
```

The config package also provides a `Required` interface. This interface lets an object indicate that configuration data is required. When the program applies configuration data to an object, if configuration data is required but it nil, it will return an error. Specifying the queue name is required because the middleware will not know where to send messages if it is unknown. Add the following to `sqs.go` to indicate our middleware requires config data.

```go
func (sm *sqsMiddleware) RequiresConfig() bool {
	return true
}
```

### Final code listing
We are now finished building the middleware. The process for building new sources and fns is very similar. You must create a struct, implement the appropriate interface, handle configuration data, and register the component with the registry.

Here is the contents of `sqs.go` in its final form.

```go
package main

import (
	"context"
	"fmt"

	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/sqs"
	"github.com/fnrun/fnrun/fn"
	"github.com/fnrun/fnrun/run"
)

type sqsMiddleware struct {
	svc      *sqs.SQS
	queueURL *string
}

func (sm *sqsMiddleware) RequiresConfig() bool {
	return true
}

func (sm *sqsMiddleware) ConfigureString(queueName string) error {
	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	sm.svc = sqs.New(sess)

	urlResult, err := sm.svc.GetQueueUrl(&sqs.GetQueueUrlInput{
		QueueName: &queueName,
	})

	if err != nil {
		return err
	}

	sm.queueURL = urlResult.QueueUrl

	return nil
}

func (sm *sqsMiddleware) Invoke(ctx context.Context, input interface{}, f fn.Fn) (interface{}, error) {
	output, err := f.Invoke(ctx, input)
	if err != nil {
		return output, err
	}

	body := fmt.Sprint(output)

	msg := sqs.SendMessageInput{
		MessageBody: &body,
		QueueUrl:    sm.queueURL,
	}
	_, err = sm.svc.SendMessage(&msg)
	if err != nil {
		return nil, err
	}

	return output, nil
}

func NewSqsMiddleware() run.Middleware {
	return &sqsMiddleware{}
}
```

## Building and running the custom runner
Now we can build our custom runner and try it out. In the [AWS SQS dashboard](https://console.aws.amazon.com/sqs) create two new queues with names `momentum-input` and `momentum-output`. Copy the following configuration into a `fnrun.yaml` in your target directory.

```yaml
source:
  fnrun.source/sqs:
    queue: momentum-input
middleware:
  - my-organization.middleware/sqs: momentum-output
  - fnrun.middleware/key: body
fn:
  fnrun.fn/http: 
    targetURL: http://localhost:30111
```

Since we are using an fn defined in our runner, we do not need to provide anything other than our runner and configuration file. Start the custom runner that best suits your environment (e.g., `./custom_fnrunner`). 

When the application is running, return to the AWS dashboard and select the `momentum-input` queue to open the its home page. Next, click the button "Send and receive messages" in the upper-right corner and enter a JSON object with mass and velocity values in the message body textarea. Finally, click the "Send message". This will enqueue a message on your queue that your custom runner will receive and process.

Now go to the home page for the `momentum-output` queue, click "Send and receive messages", and then click the "Poll for messages" button. You should see a new message appear. If you click on the link on the message ID, the console will show you the body for the message. It should contain a JSON message containing the calculated momentum value.

## Conclusion
Congratulations! You have reached the end of the fnrun tutorial. Throughout this tutorial, you learned about the types of components fnrun provides and how they worked together. You also learned many of the specific components that ship with fnrun. You learned how to compose these pieces into fnrun applications and how to deploy those applications. Finally, you learned how to extend fnrun to meet unique business needs.

Thank you for taking the time to work through the tutorial. We hope you found it useful. If you had issues or if you have any questions or ideas, please let us know. You can get in touch by opening an issue or starting a discussion on GitHub or by sending an email to [hello@fnrun.dev](mailto:hello@fnrun.dev).