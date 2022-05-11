---
title: "Improve & Repeat"
chapter: false
weight: 30
services: true
---


## Learning and Improving

In the previous section we have identified that our ECS task doesn't capture progress: Every time it's interrupted, it starts from scratch. Now we want to improve the application and re-run the FIS experiment.

### Improve the application

We want to change the application, such that it reads from the DB **first** to understand where it should continue.

**If it's the first run,** it will find no entries in the DB and therefore start processing at index 0.

**If it's a subsequent run,** it will keep iterating through the DB (index 0,1,2,3...) and find the point where it was interrupted.

*This approach is extremely simplistic and certainly not production-ready, but it serves the purpose for this example.* 


#### Improved code
We have prepared a second ECS Task Definition using an improved Docker container image for you to use. Here is an excerpt from the code for you to understand.

The old main-loop looked like this:
```javascript
while(index < batchSize){
    //do some work. Write results and index to DynamoDB.
    index++;
}
```

While the new main-loop of the application looks like:

```javascript
continueAt()
.then(cont => {
  console.log("Continuing at ", cont)
  while(cont < batchSize){
    //do some work. Write results and index to DynamoDB.
    cont++;
  }  
}).catch(err => console.log('Failed execution of batch job', err))
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

 - Switch to the FIS console
 - Select the `FisWorkshopECSTask` experiment template we created before
 - Click **Start experiment** 
 - Start the experiment

 ### Check progress

 On the ECS task overview, check the **Logs** tab again. Again you should see some log messages showing progress.

 ++++ TODO Screenshot Logs with progress +++

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

 On the task overview, click on **Logs** again. You should see something like the following:

++++ TODO screenshot showing logs where the task has started a second time and continued at an index != 0 +++

You can see how the improved task was interrupted, then re-started and instead of restarting from index 0, it uses additional logic to prevent duplicate work. Great!

## ECS further learning

For more on ECS configurations see the [ECS workshop](https://ecsworkshop.com/).
