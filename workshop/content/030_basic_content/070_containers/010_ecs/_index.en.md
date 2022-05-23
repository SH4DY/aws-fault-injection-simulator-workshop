---
title: "Amazon ECS"
chapter: false
weight: 10
services: true
---
Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that helps you easily deploy, manage, and scale containerized applications. It deeply integrates with the rest of the AWS platform to provide a secure and easy-to-use solution for running container workloads in the cloud and now on your infrastructure with Amazon ECS Anywhere.

In this section you will find two experiments. In the first you will spin up a web service on ECS and use FIS to explore and improve its resilience. In the second, you will work with an invoked ECS task that represents something like a recurring batch job that needs to be restarted by external automation on any failure. You will use FIS to simulate a failure by stopping the task, and use the learnings to improve overall performance on the task.