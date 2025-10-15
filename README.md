### aws_lambda
+ **Ec2 computing:**
  + It's a server-based (like EC2) computing.
  + When we want to run an application, we will create an ec2 instance in aws by providing details like AMI, instance type, etc. Then instance is created and we can manage it.
  + An ip is attached to that instance and we have the full control of the OS, filesystem, network, packages.
  + We can SSH into it and make configurations.
  + When our task is done we can destroy the instance. 
  + Here, we manage the infrastructure.
+ **Aws lambda:**
  + It's a serverless computing.
  + When we create a Lambda function, we are not creating or managing any server.
  + AWS will run your code in a temporary, managed container.
  + we don’t see or control the underlying server, OS, or IP. we cannot SSH into it as we don't know the IP.
  + The environment exists only for a few seconds or minutes — then it’s destroyed automatically.
  + Here, we don't manage the infrastructure. So, using lambda we can save the cost.
  + **The best use case of serverless architecture is cost optimization** 
  + Why we don't get the details of the server ?
    + The infrastructure is abstracted away (AWS manages it for you). which means we need to care about the infra, aws will manage it and destroy it when task is done.
    + Multiple customers’ Lambdas might share the same underlying infrastructure.
    + The server our function runs on might be different each time we invoke it.
    + The environment is ephemeral — created and destroyed as needed.
  + But sometimes apps require fixed IPs then AWS lets us to run Lambda inside a VPC (Virtual Private Cloud). But even then, we still don’t get SSH or OS access — only network-level control.
  + Lambda never runs on its own. It waits for an event — and when that event happens, AWS spins up an execution environment, runs your function, and then tears it down.
  + Lambda scales up and down automatically to handle your workloads, and we don't pay anything when our code isn't running.
  + An event is simply a trigger or signal telling Lambda to run your code.
      | Source                              | Event Type        | What it means                            |
      | ----------------------------------- | ----------------- | ---------------------------------------- |
      | **API Gateway**                     | HTTP request      | Someone hit your API endpoint            |
      | **S3**                              | Object created    | A new file was uploaded                  |
      | **CloudWatch Events / EventBridge** | Scheduled event   | Run at a specific time (like a cron job) |
      | **SNS**                             | Message published | A notification message arrived           |
      | **DynamoDB Streams**                | Table update      | A row was inserted/modified              |
  + AWS Lambda = Code that runs only when triggered by an event.
  + CloudWatch / EventBridge = Scheduler that generates the event.

+ **In EC2 — you manage and know the server.**
+ **In Lambda — AWS manages and hides the server.**
  + We just provide code and triggers; AWS handles the rest.
+ `def lambda_handler()` is the function which is called when an event is triggered. It’s like the `main()` function in C or Java.
  + It’s the entry point AWS Lambda uses to start your code.
  + If there are 10 functions in the code then all those functions need to be declared in that function.
  + So, Lambda calls only one function — the handler — when triggered
  + Any other functions must be called from inside that handler.
  + We can change to other name if required by editing the Lambda configuration,

+ For ebs snapshots we usigng `ec2 = boto3.client('ec2')` because
  + When we give that aws_lambda connects to the EC2 service API — basically, the AWS backend that manages everything under the “EC2” umbrella (instances, volumes, snapshots, AMIs, etc.)
  + EBS is part of the EC2 service family in AWS. There’s no separate service called “EBS” in boto3
  + So, anything you want to do with volumes or snapshots, you must call the EC2 API (since AWS groups EBS operations under EC2)
  + To talk to EBS (for listing, creating, or deleting snapshots), Lambda must use the EC2 API client via boto3.client('ec2')

+ aws_lambda function for deleting stale snapshost which are not assigned to any volumes or attached volumes are not inuse by any instance
  + When we try to describe the snapshots, volumes, instances it will throw an error like it's not accessible as no permission to access it.
  + By default aws lamdba creates a roles for function or we can select existing role.
    + That role should have a policy to describe the instances, volumes & snapshots. 
