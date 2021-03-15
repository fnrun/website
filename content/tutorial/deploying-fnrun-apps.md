---
title : "Deploying fnrun applications"
description: "Learn how to deploy fnrun applications with examples for k8s and AWS Lambda"
lead: ""
date: 2021-03-15T18:45:00+00:00
lastmod: 2021-03-15T18:45:00+00:00
draft: false
images: []
menu: 
  tutorial:
    parent: "tutorial"
toc: true
weight: 112
---

In Part 1 of the tutorial we an fnrun application that calculated momentum and provided a simple JSON API. In Part 2 will look at the basic deployment requirements for fnrun applications and demonstrate how to deploy our momentum calculator to Kubernetes and AWS Lambda.

## Basic deployment requirements
There are four basic requirements for fnrun applications. They include an fnrun runner and associated configuration, a business function, and the environment for running the business function.

A runner is an application that creates the application infrastructure and runs the business function. fnrun ships an official runner called `fnrunner` with each release. fnrun provides builds of fnrunner for Linux, macOS, and Windows, and these executables do not need any additional runtime. fnrunner consumes YAML configuration data from files. By default it will look for a file called `fnrun.yaml`, but you may override this behavior by using the `-f` flag or setting an environment variable called `CONFIG_FILE` and providing a path to the desired config file. It is worth noting that organizations may create custom runners that have different behaviors than fnrunner, and they might not require configuration data.

The configuration data to be provided with fnrunner was discussed in Part 1. In short, it provides instructions to fnrunner about how to build the application infrastructure necessary to process inputs and outputs around a business function.

The business function is the implementation of business logic. It is the most important part of an fnrun application and is written and maintained by developers in your organization.

Finally, the deployment environment must contain the necessary runtime for the business function itself. In our example, we have created a statically compiled executable from the Fortran code, so we do not have any additional requirements for application runtime. However, if our application had been written in Java, we would need a JVM installed in our deployment environment.

## Deploying to Kubernetes
For this section, you will need Kubernetes running locally. Any easy option is to enable Kubernetes in Docker Desktop. Otherwise, you may consider using minikube to create a local k8s cluster.

Since we created a Docker image in Part 1, it is easy to deploy to Kubernetes. Copy the following configuration into a new file called `k8s.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fnrun-tutorial-deployment
  labels:
    app: fnrun-tutorial
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fnrun-tutorial
  template:
    metadata:
      labels:
        app: fnrun-tutorial
    spec:
      containers:
        - name: fnrun-tutorial
          image: fnrun-tutorial:0.1.0
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
          env:
            - name: PORT
              value: "8080"

---

apiVersion: v1
kind: Service
metadata:
  name: fnrun-tutorial-service
spec:
  selector:
    app: fnrun-tutorial
  type: NodePort
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30111
```

This configuration creates a deployment that will run our application and creates a service that will expose our application on port 30111. Deploy to k8s using the following command: `kubectl apply -f ./k8s.yaml`. After the pod starts up, you should be able to interact with the application as you did in Part 1.

## Deploying to AWS Lambda
Next, we will deploy our application to AWS Lambda. Lambda provides several mechanisms for deploying functions. We will deploy to a Lambda with a custom runtime.

### Preparing the function
To support this scenario, fnrun provides a source for lambda. The source will deserialize an event body from JSON by default and provide it to the middleware pipeline. Copy the following into a file called `lambda.yaml`. We will use this as our fnrunner configuration file.

```yaml
source: fnrun.source/lambda
middleware:
  - fnrun.middleware/key: body
  - fnrun.middleware/json:
      output: serialize
  - fnrun.middleware/jq:
      input: '[.mass, .velocity] | join(",")'
      output: '{ "momentum": split(",") | .[2] | tonumber }'
fn:
  fnrun.fn/cli: ./momentum
```

> Note: You can deploy a Docker-based AWS Lambda function using the same configuration listed above. We will upload a zip file simply to show a variety of ways to deploy. Interestingly, if you package both the `fnrun.yaml` and `lambda.yaml` fnrun configurations into the same Docker image, you can run in both environments with the same image.

When using a custom runtime, Lambda expects to have an executable file called bootstrap to start the function execution. Create a file called `bootstrap` and copy in the following contents. Note that all the bootstrap is doing is starting fnrunner and telling it to read the `lambda.yaml` file for configuration. After you have created this file, make sure that it is executable by running the command `chmod +x ./bootstrap` in your terminal.

```bash
#!/bin/sh
./fnrunner -f lambda.yaml
```

Next, we need to get the executables from our Docker image so we can provide them to Lambda. Use the following script to extract the momentum and fnrunner executables.

```bash
docker create -ti --name dummy fnrun-tutorial:0.1.0 bash
docker cp dummy:/momentum momentum
docker cp dummy:/fnrunner fnrunner
docker rm -f dummy
```

Finally, create a zip file containing the momentum executable, fnrunner, the lambda.yaml configuration file, and the bootstrap script. You can do this at the terminal with the following command.

```bash
zip lambda.zip ./momentum ./fnrunner ./lambda.yaml ./bootstrap
```

### Creating and deploying the function
Visit the [AWS Lambda dashboard](https://console.aws.amazon.com/lambda) and click the 'Create function' button. The console will display a page that gives you options for the function. Make sure the 'Author from scratch' option is selected, and then provide a function name (e.g., `fnrunTutorial`) and select the runtime option labeled 'Provide your own bootstrap on Amazon Linux 2'.

On the function page, click the button labeled "Upload from" and choose the option ".zip file" to upload the zip file we created to Lambda. 

When the upload completes, open the "Test" tab, enter a JSON payload with mass and velocity values, and press the "Invoke" function. You should see a green box appear with the text "Execution result: succeeded". If you expand the message, you should see the calculated momentum values in the details.

## Conclusion
In this part of the tutorial we learned the four requirements for deploying fnrun applications and then practiced deploying our momentum calculator to both Kubernetes and AWS Lambda. Since fnrun ships executables for a number of platforms, you can create similar deployments in other environments.

So far, we have been focused on using the official fnrunner. However, there are times when fnrunner does not provide all the functionality you need. In the next part, we will extend fnrun and create a custom runner that helps us solve a new problem. However, the requirements for deployment will not change, and you would be able to deploy a new system using the techniques you learned in this part of the tutorial.