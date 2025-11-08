### aws_lambda
+ **Ec2 computing:**
  + It's a server-based (like EC2) computing.
  + When we want to run an application, we will create an ec2 instance in aws by providing details like AMI, instance type, etc. Then instance is created and we can manage it.
  + An ip is attached to that instance and we have the full control of the OS, filesystem, network, packages.
  + We can SSH into it and make configurations.
  + When our task is done we can destroy the instance. 
  + Here, we manage the infrastructure.
+ **Aws lambda:**
  + Boto3, the official AWS SDK for Python, is used to create, configure, and manage AWS services.
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

#### client vs resources:
+ boto3 internally uses the AWS SDK for Python to make HTTPS API calls to AWS services.
+ Boto3 offers two distinct ways of accessing these abstracted APIs
  + Client: low-level service access
  + Resource: higher-level object-oriented service access
+ **client:**  
  + `ec2_Client =  boto3.client(ec2)`
  + In boto3, the client represents a low-level interface to a specific AWS service.
  + Means, Clients allow you to make direct calls to AWS service APIs
  + The response you get is a dictionary (JSON) — full of nested keys.
  + it gives you full control but requires working with raw JSON responses.
  + Clients require explicit configuration and credentials. You need to provide AWS access and secret keys, and optionally, session tokens and region information.
  + Best for automation scripts, fine-grained control, or when you want to call exact AWS API actions.
+ **Resource:**
  + `ec2_resource = boto3.resource('ec2')`
  +  Boto3 resources, on the other hand, provide a higher-level, object-oriented interface to AWS services.
  +  Resources encapsulate the underlying API calls and provide a more Pythonic and user-friendly experience.
  +  You’re not dealing with raw JSON data.
  +  It’s easier to use and more readable, but doesn’t expose every single API operation.
+  **Analogy:**
  + Think of boto3 (the AWS SDK for Python) like a remote control for AWS
    + The client is like pressing raw buttons on a remote — you have to know exactly what command to send (it’s lower-level).
    + The resource is like a smart remote — you just say what you want, and it figures out the details for you (it’s higher-level, object-oriented).
    + Both control AWS — but they do it differently.
  + | Concept      | Analogy                                                                                | Example                                                      |
    | ------------ | -------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
    | **Client**   | Talking to AWS using raw API calls (you must know exact command names & data formats). | “Call the EC2 API and give me all instance details as JSON.” |
    | **Resource** | Talking to AWS in plain Python objects (simpler, human-readable).                      | “Hey AWS, list all EC2 instance objects for me.”             |
       

#### steps to program:
+ First, the function name should be `lambda_handler()`
+ mention the client,like ec2 = boto3.client('ec2')
  + If we want to work with EC2 → use ec2 client
  + If we want to work with S3 → use s3 client
  + If we want to work with volumes → also ec2 client (because EBS is part of EC2)”
+ He wrote the script like, first we already have running instances, will stop those instances first and later will start those stopped instances using the lambda function
+ **How it works in companies for stopping and creating instances:**
  + You tag instances with something like: `Auto-instance-scheduler=yes`
  + One Lambda function runs at 8 AM to start all tagged instances
  + Another Lambda function runs at 8 PM to stop them
  + Both functions are triggered via CloudWatch Events (cron)
  + This way, developers have access to their dev/QA servers during working hours only. At night, those instances are stopped → saving compute cost.  
  + **Cost calculation:**
    + When you stop an EC2 instance:
      + The instance shuts down like a normal computer.
      + Its EBS (root) volume — i.e., your disk — remains attached (and data stays intact).
      + The instance ID and configuration are preserved.
      + The compute (CPU/RAM) resources are released back to AWS — meaning you are no longer billed for compute time.
      + **So you stop paying for:**
        + EC2 compute hours (the biggest cost for running instances)
      + ❌ **But you still pay for:**
        + EBS storage (because the volume still exists)
        + Elastic IP (if it’s not attached to a running instance)
        + Snapshots / other attached storage
      + But since **compute is usually the most expensive part**,you can still save 60–80% of the total EC2 bill by stopping instances overnight. 
-----
#### What happens when a Lambda runs?
+ Every time your Lambda function executes, AWS automatically sends logs to **CloudWatch Logs**.
+ But for Lambda to do that, it needs permissions to create and write to CloudWatch Log Groups and Streams.
+ When your Lambda runs for the first time, it needs to:
  + Create a new Log Group → /aws/lambda/<function-name> (if it doesn’t exist already)
  + Create a Log Stream → something like /aws/lambda/my-lambda/2025/11/06
  + Put log events → actually write the log lines (your function’s output)
+ Without these, Lambda would still run but fail to push logs — making debugging impossible.”
---
+ For ebs snapshots we usigng `ec2 = boto3.client('ec2')` because
+ When we give that aws_lambda connects to the EC2 service API — basically, the AWS backend that manages everything under the “EC2” umbrella (instances, volumes, snapshots, AMIs, etc.)
+ EBS is part of the EC2 service family in AWS. There’s no separate service called “EBS” in boto3
+ So, anything you want to do with volumes or snapshots, you must call the EC2 API (since AWS groups EBS operations under EC2)
+ To talk to EBS (for listing, creating, or deleting snapshots), Lambda must use the EC2 API client via boto3.client('ec2')

+ aws_lambda function for deleting stale snapshost which are not assigned to any volumes or attached volumes are not inuse by any instance
  + When we try to describe the snapshots, volumes, instances it will throw an error like it's not accessible as no permission to access it.
  + By default aws lamdba creates a roles for function or we can select existing role.
    + That role should have a policy to describe the instances, volumes & snapshots. 

+ Your Lambda didn’t finish executing before the timeout. This is a common issue when:
  + You’re calling AWS APIs like describe_snapshots, describe_volumes, etc., and the response takes longer than expected.
  + If you have hundreds or thousands of snapshots/volumes, this can be slow.
  + Your Lambda’s timeout setting is too low. **Default is usually 3 seconds**, which is clearly too short for EBS snapshot operations.
  + Sometimes network issues in Lambda can slow API calls, especially for multiple describe_volumes calls in a loop.
  + fix: Increase Lambda timeout --> Go to Lambda → Configuration → General configuration → Timeout → increase to 30 seconds or more.
