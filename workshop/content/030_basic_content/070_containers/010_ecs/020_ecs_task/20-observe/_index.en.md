---
title: "Observe the processing job"
chapter: false
weight: 20
services: true
---

## Restart the processing job

Now that we have interrupted our processing job, we want to restart it and observe how it eventually finishes writing all items to the database. We can do that by spinning up a new task with the same container image and then check logs.

 - For this purpose, navigate to the "Elastic Container Service" dashboard.
 - Select the `FisStackECS-Cluster`
 - Click **Tasks** and **Run new task**
 - Select `FARGATE` and `Linux`
 - For "Task Definition" select the `FisStopTaskStackTaskDef`
 - For **Cluster VPC** select the one ending with `...FisVpc` 
 - Select one of the 2 **Private Subnets** - we made sure they have a route to the internet to retrieve the Docker image
 - At the bottom of the page, make sure the tag `fisTarget` - `true` is already predefined. FIS uses this tag to target.
 - Finally click "Run Task"

### Observations

After a few minutes you can already start observing how the restarted task runs and writes items into the database.

 - Navigate to the [AWS Cloudwatch console](console.aws.amazon.com/cloudwatch/home?region=us-west-2).
 - Click on **Log Groups**
 - Use the search bar to look for a group starting with `FisStopTask`. Click on it.
 - Click **Search log group**
 - This view shows all logs in a certain time frame that this ECS task published.

After some minutes you will see a log message indicating that the task completed its work and wrote the last item to the database, **index 9**.

TODO Replace with screenshot that shows completion
 {{< img "task-cloudwatch-log-ungraceful.png" "Log Group in CloudWatch showing task started from 0 after interrruption" >}}

Unfortunately, you will see that the task has no notion of "checkpointing" and it wasn't able to capture it's progress from the first run. It simply started from **index 0**. In the next section we'll improve the app.