---
title: "Hypothesis & Experiment"
chapter: false
weight: 10
services: true
---
 
## Experiment idea - Graceful degradation

In this experiment we want to understand how our processing task reacts to being interrupted. You will start a task that does something (like a calculation) and writes results into a database (DynamoDB). If a task is interrupted before writing all calculation results into the database, it should remember it's progress and continue where it left off when it's restarted. This is sometimes called *graceful degradation* because we anticipate failure and try to improve behavior in the case of interruption.

* **Given**: we have a processing job running as an ECS task
* **Hypothesis**: If the task is interrupted before finishing and we restart it, it should NOT start from scratch but rather understand where it left off and continue at that point.

## Experiment setup

{{% notice note %}}
We are assuming that you know how to set up a basic FIS experiment and will focus on things specific to this experiment. If you need a refresher see the previous [**First Experiment**]({{< ref "030_basic_content/030_basic_experiment/" >}}) section.
{{% /notice %}}

### General template setup

{{% notice note %}}
We are assuming that you have already set up an IAM role for for this workshop. If you haven't, see the [**Create FIS Service Role**]({{< ref "030_basic_content/030_basic_experiment/10-permissions" >}}) section.
{{% /notice %}}

Create a new experiment template:
  * add `Name` tag of `FisWorkshopECSTask`
  * add `Description` of `Stop ECS Task during processing`
  * select `FisWorkshopServiceRole` as execution role



### Action definition
For "Name" enter `StopECSTask` and you can skip the Description. For "Action type" select `aws:ecs:stop-task`. Click "Save".

### Target selection

We will skip target selection for now and come back to it once we start the ECS task.

### Creating template without stop conditions

Confirm that you wish to create the template without stop condition.


## Validation procedure - Run the task successfully

Now let's run the task, which will take around **5 minutes** to complete. It writes an item to DynamoDB every **30 seconds**

 - For this purpose, navigate to the "Elastic Container Service" dashboard.
 - Select the `FisStackECS-Cluster`
 - Click `Tasks` and `Run new task`
 - Select `FARGATE` and `Linux`
 - For "Task Definition" select the `FisStopTaskStackTaskDef`
 - For "Cluster VPC" select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - Finally click "Run Task"
 - Lastly, take note of the task ID that should looke like '96a7f974215d4c5cad9f57a64431722a'

## Run FIS experiment - Interrupt the task

In this 5 minute window, we now want to interrupt the task with FIS:

 - Switch to the FIS console
 - Select the `FisWorkshopECSTask` experiment template we created before
 - Click "Actions" - "Update" 
 - In the "Targets" section, click "Edit"
 - Under "Resource IDs", enter or select the task ID you noted above.
 - Hit "Save" and "Update experiment template"
 - Start the experiment


### Checking results of processing in CloudWatch

Back on the cluster overview

 - Click `Tasks'
 - Click on the task id of the task you just started
 - Here you can see if your tasks is already RUNNING or in PENDING state
 - Change to the Logs tab. Alternatively you can navigate to CloudWatch Logs and find your container logs there.
 - The task we are running logs every processing step and a few minutes after starting the task, you should see an output similar to the following:

 ++++ TODO Screenshot task logs ++++ 

You can see how the ECS task wrote an incrementing index to DynamoDB every 30 seconds but FIS stopped the task before writing all items.

By the way: Feel free to navigate to the DynamoDB page and check the table `FisStopTaskStack-DynamoTable...`. You will see the items the ECS task wrote.