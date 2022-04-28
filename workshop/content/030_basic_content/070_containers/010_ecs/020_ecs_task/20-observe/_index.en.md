---
title: "Observe the processing job"
chapter: false
weight: 20
services: true
---

## Restart the processing job

Now that we have interrupted our processing job, we want to restart it and hope that it has some sort of ability to start where it left off. We can do that by spinning up a new task with the same container image and then check logs.

 - For this purpose, navigate to the "Elastic Container Service" dashboard.
 - Select the `FisStackECS-Cluster`
 - Click `Tasks` and `Run new task`
 - Select `FARGATE` and `Linux`
 - For "Task Definition" select the `FisStopTaskStackTaskDef`
 - For "Cluster VPC" select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - Finally click "Run Task"
 - Lastly, take note of the task ID that should looke like '96a7f974215d4c5cad9f57a64431722a'

### Observations

Give it 1-2 minutes and then check the logs:

 - On your ECS cluster overview, click `Tasks'
 - Click on the task id of the task you just started
 - Change to the Logs tab. 

 +++ TODO Screenshot that shows "Starting processing job at index 0 +++

 Unfortunately, you will see that the task has no notion of "graceful degradation" and it wasn't able to capture it's progress from the first run. It simply started from **index 0**. 

We could make multiple improvements to the current state:

 1. The app inside the container can be improved to *read* the current state (i.e. which indexes were already written into the DB) and continue it's work from there.
 2. You could implement a re-start mechanism through an independent process. This process could frequently check the health of the processing job and the progress in the database. If needed it could restart the task. You could use [Amazon EventBridge](https://aws.amazon.com/eventbridge/) and [AWS Lambda](https://aws.amazon.com/lambda/) for this purpose.
 3. Alternatively, consider a re-architecture where an [Amazon SQS](https://aws.amazon.com/sqs/) queue is used to contain every work item. The app could simply poll one item after another and process them. If the app is interrupted, the queue is carries state implicitly.

In this workshop we will focus on **improvement area 1**.