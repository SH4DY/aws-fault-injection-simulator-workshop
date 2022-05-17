---
title: "Amazon ECS Tasks"
chapter: false
weight: 20
services: true
---


In this section we will cover working with containers running on [**Amazon Elastic Container Service**](https://aws.amazon.com/ecs/) (ECS) as a task. In contrast to services, tasks are used for processing a clearly defined workload, like a batch job. A task spins up, does a specific amount of work, and completes as soon as the CMD script of your Dockerfile terminates.

{{< img "ecs-task-cluster.png" "Image of architecture to be injected with chaos" >}}