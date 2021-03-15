---
title : "Introduction"
description: "The introduction to the fnrun tutorial"
lead: ""
date: 2021-03-15T18:45:00+00:00
lastmod: 2021-03-15T18:45:00+00:00
draft: false
images: []
menu: 
  tutorial:
    parent: "tutorial"
weight: 110
toc: true
---

Welcome to the fnrun tutorial. This tutorial will teach you everything you need to know to use fnrun. It has three parts. The first part focuses on building business functions with fnrun. The second part discusses how to deploy fnrun applications. The final part explores how you can extend fnrun to meet your organization's unique needs.

> We prefer the term "business function" to "serverless function" for a couple of reasons. First, fnrun does not care how you deploy. It is compatible with server-based and serverless environments. Second, fnrun separates deployment and application infrastructure concerns from business logic. This enables developers to focus on building business value. The term "business function" captures the same intent as "serverless function" without needing to argue over definitions.

fnrun is a system that aims to help developers focus on building business value instead of application infrastructure such as HTTP servers and queue clients. To achieve this, fnrun provides a system for describing infrastructure and defining contracts for calling business functions. The default contract is simple: a business function must read input from stdin and write output to stdout.

fnrun is inspired in part Kelsey Hightower's 2018 KubeCon Keynote - [Kubernetes and the Path to Serverless](https://www.youtube.com/watch?v=oNa3xK2GFKY). It is a fantastic talk in which Kelsey encourages us to think about how we can help developers focus on the business task at hand. Everything outside that task is noise. 

To demonstrate this idea, Kelsey developed a simple Fortran program to calculate momentum and showed how this program could be used in a couple of different contexts. He ran the program on Kubernetes and then packaged the same program and ran it on AWS Lambda. He built shims to wire the application into its environment and feed it data.

These ideas are central to fnrun. In this tutorial, we will start with the same program Kelsey used to calculate momentum and write an fnrun configuration file to build a JSON API around it. Next, we will learn how to deploy fnrun applications to both Kubernetes and AWS Lambda. Finally, we will explore the limits of what fnrun provides and learn how to extend it to meet a new business need.

You will need Docker installed and running locally as well as a Kubernetes cluster you can deploy into. I use the one that ships with Docker Desktop. You will also need an AWS account. Everything we do in this tutorial will be available under the AWS Free Tier.