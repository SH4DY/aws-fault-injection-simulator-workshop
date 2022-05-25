---
title: "Hypothesis & Experiment"
chapter: false
weight: 10
services: true
---
 
## Experiment idea - Checkpointing

In this experiment we want to understand how our processing task reacts to being interrupted. You will start a task that does something (like a calculation) and writes results into a database (DynamoDB). If a task is interrupted before writing all calculation results into the database, it should be restartable and eventually complete all processing items.

* **Given**: we have a processing job running as an ECS task
* **Hypothesis**: If the task is interrupted before finishing, it can be restarted and will correctly write all processing items into the database.

Ideally it should even remember it's progress and continue where it left off when it's restarted. This is sometimes called *checkpointing* because we anticipate failure and try to improve behavior in the case of interruption. This concept will be introduced as a second experiment.


## The task

An ECS Task is a Docker container instance. In this experiment we will run it on [AWS Fargate](https://aws.amazon.com/fargate/?nc1=h_ls), which is a **serverless compute engine**. That means you can write a Docker-based application and run it on ECS without having to deal with underlying EC2 instances. 

In this example, our task is a simple Node.js application that *slowly* writes an increasing integer index into a [Amazon DynamoDB](https://aws.amazon.com/dynamodb) table. It simulates an app that processes a number of items and writes results into a database.

TODO link to app code

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

 - In the "Targets" section, click **Edit**
 - Select **Resource tags, filter and parameters"
 - We will use a tag to identify the task. The tag is defined in the ECS Task definition and you wil be able to see it when you run the task.
 - Under **Resource tags** enter `fisTarget` for **Key** and `true` for **Value**
 - Click **Save**

### Creating the template without stop conditions

Click **Create experiment template** and confirm that you wish to create the template without stop condition.

## Validation procedure - Run the task successfully

Now let's run the task, which will take around **5 minutes** to complete. It writes an item to DynamoDB every **30 seconds**

 - For this purpose, navigate to the "Elastic Container Service" dashboard.
 - Select the `FisStackECS-Cluster`
 - Click `Tasks` and `Run new task`
 - Select `FARGATE` and `Linux`
 - For "Task Definition" select the `FisStopTaskStackTaskDef`
 - For "Cluster VPC" select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - At the bottom of the page, make sure the tag `fisTarget` - `true` is already predefined. FIS uses this tag to target.
 - Finally click "Run Task"

{{% notice note %}}
This experiment is not designed to be run in parallel with multiple instances of the same ECS task. Please only launch 1 instance. 
Later in this chapter we will discuss improvement ideas that can make this workload parallelizable.
{{% /notice %}}

## Run FIS experiment - Interrupt the task

In this 5 minute window, we now want to interrupt the task with FIS:

 - Switch to the FIS console
 - Select the `FisWorkshopECSTask` experiment template we created before
 - Click **Start experiment** 
 - Start the experiment


### Checking results of processing in CloudWatch

We can now check the container logs to understand at which point the task was interrupted.

 - Navigate to the [AWS Cloudwatch console](console.aws.amazon.com/cloudwatch/home?region=us-west-2).
 - Click on **Log Groups**
 - Use the search bar to look for a group starting with `FisStopTask`. Click on it.
 - Click **Search log group**
 - This view shows all logs in a certain time frame that this ECS task published.

 TODO update screenshot below
{{< img "task-log-interrupted.png" "ECS Task log output interrupted by FIS" >}}

You can see how the ECS task wrote an incrementing index to DynamoDB every 30 seconds but FIS stopped the task before writing all items.

By the way: Feel free to navigate to the DynamoDB page and check the table `FisStopTaskStack-DynamoTable...`. You will see the items the ECS task wrote.