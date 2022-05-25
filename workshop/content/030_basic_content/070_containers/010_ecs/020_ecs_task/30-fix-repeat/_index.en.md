---
title: "Improve & Repeat"
chapter: false
weight: 30
services: true
---

## Improvement ideas 

We could make multiple improvements to the current state of the application:

 1. The app inside the container can be improved to *read* the current state (i.e. which indexes were already written into the DB) and continue it's work from there.
 2. You could implement a re-start mechanism through an independent process. This process could frequently check the health of the processing job and the progress in the database. If needed it could restart the task. You could use [Amazon EventBridge](https://aws.amazon.com/eventbridge/) and [AWS Lambda](https://aws.amazon.com/lambda/) for this purpose.
 3. Alternatively, consider a re-architecture where an [Amazon SQS](https://aws.amazon.com/sqs/) queue is used to contain every work item. The app could simply poll one item after another and process them. If the app is interrupted, the queue is carries state implicitly.

In this chapter we will focus on **improvement area 1**. This will allow us to save the time needed to re-process items that were already written to the database.

Improvement **2** would further eliminate manual effort, as it could automatically restart an interrupted process.

Improvement **3** is an elegant way to allow parallelization of processing, as the queue simply holds all work items and multiple containers can pick up work items from the queue.

## Checkpointing

In the previous section we have identified that our ECS task doesn't capture progress: Every time it's interrupted, it starts from scratch. 

The idea of checkpointing is to save processing time (and consequently cost) whenever our task is interrupted. If a complete run would take hours and the task is interrupted, we want it to restart at the index it was interrupted at.

### Improve the application

We want to change the application, such that it reads from the DB **first** to understand where it should continue.

**If it's the first run,** it will find no entries in the DB and therefore start processing at index 0.

**If it's a subsequent run,** it will keep iterating through the DB (index 0,1,2,3...) and find the point where it was interrupted.

*This approach is very simplistic and scales linearly with the number of items but it serves the purpose of this example. Refer to the section above for improvements that could make such a workload production ready.* 

### Improved code
We have prepared a second ECS Task Definition using an improved Docker container image for you to use. Here is an excerpt from the code for you to understand.

The old main-loop looked like this:
```javascript
console.log("Starting at ", index)
while(index < batchSize){
    //do some work. Write results and index to DynamoDB.
    index++;
}
```

While the new main-loop of the application looks like:

```javascript
//Before going into the work-loop, we call a function that reads from the database and returns the highest found index previously written
continueAt()
.then(index => {
  console.log("Starting at ", index)
  while(cont < batchSize){
    //do some work. Write results and index to DynamoDB.
    index++;
  }  
})
```

The key being the `continueAt()` function that is called first, reading from the DB and returning with the index to continue with `cont`. This is also what you will see in log output.

+++ TODO link to code in GitHub for readers that are interested +++

### Clear the DynamoDB table

Before we re-run the experiment we need to clear the DynamoDB table from the results of the previous run.

 - Navigate to the [DynamoDB console](console.aws.amazon.com/dynamodbv2)
 - Click on **Tables** and on the table starting with **FisStopTask...**
 - Click **Explore table items**
 - After clicking **Run**, you can select the uppermost checkbox of items to select all of them
 - In the **Actions** dropdown menu, click **Delete** and confirm

## Repeat the experiment

Now we want to re-run the task and repeat the experiment:

 - Navigate to the [ECS console](console.aws.amazon.com/ecs/home) 
 - Click **Task Definitions** and select `FisStopTaskImproved...` which contains the improved code
 - Click **Actions** and **Run Task**
 - Select the `FisStackECS-Cluster`
 - Click **Tasks** and **Run new task**
 - Select `FARGATE` and `Linux`
 - For **Cluster VPC** select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - Finally click "Run Task"

Now let's interrupt the task again:

 - Switch to the [FIS console](console.aws.amazon.com/fis/home)
 - Select the `FisWorkshopECSTask` experiment template we created before
 - Click **Start experiment** 
 - Start the experiment

 ### Check progress

 On the ECS task overview, check the **Logs** tab again. Again you should see some log messages showing where the task was interrupted.

 {{< img "task-log-interrupted.png" "ECS Task log output showing where task was interrupted" >}}

 So far, so good.

 ### Restart the task

Now let's restart the improved task and check if it actually continues where it left off.

- Navigate to the [ECS console](console.aws.amazon.com/ecs/home) 
 - Click **Task Definitions** and select `FisStopTaskImproved...` which contains the improved code
 - Click **Actions** and **Run Task**
 - Select the `FisStackECS-Cluster`
 - Click **Tasks** and **Run new task**
 - Select `FARGATE` and `Linux`
 - For **Cluster VPC** select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - Finally click "Run Task"

 ### Observe results

Let's check the results

 - Navigate to the [AWS Cloudwatch console](console.aws.amazon.com/cloudwatch/home?region=us-west-2).
 - Click on **Log Groups**
 - Use the search bar to look for a group starting with `FisStopTask`. Click on it.
 - Click **Search log group**
 - This view shows all logs in a certain time frame that this ECS task published.

+++ TODO replace screenshot +++
{{< img "task-log-continued-at.png" "ECS Task log output showing task started where it left off when interrupted" >}}


You can see how the improved task was interrupted, then re-started and instead of restarting from index 0, it uses additional logic to prevent duplicate work. Great!

## ECS further learning

For more on ECS configurations see the [ECS workshop](https://ecsworkshop.com/).
