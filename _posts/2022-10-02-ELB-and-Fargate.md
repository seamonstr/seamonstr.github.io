h1. Getting a basic ELB running with a Fargate-hosted service

Goal is to write the Terraform for the ELB, its security group, listener and target group, and get those created on AWS. No config or anything; just a hello world.

Installing Terraform requires adding the Hashicorp repo and its associated keys; docs on Hashicorp's site.

Decided to add just a basic ELB with a Fargate service and a "Hello world" http server to get something working. 

Initialised a new Terraform project:
* Add a "main.tf" with the AWS provider in it:
```provider aws {
	region = "eu-west-2"
}```
* Added an elb.tf

Inet-facing ELBs need to be configured to accept traffic from the internet on their listener port, and permitted to send traffic onward to their targets' port. They also need access to their targets' health ports. This is done by creating a security group(s) for the ELB.

ELBs need to have subnets provided, with more than one AZ represented. It will use the subnets to allocate itself more IP addresses in as it scales with traffic.

List comprehensions in Terraform: `subnets = [for subnet in local.subnet.ids: subnet.id]`

ELBs can write access logs to a specified S3 bucket. It doesn't seem to be documented what security policy the bucket needs to have, or who the ELB will be writing as.

Installed the AWS CLI from Amazon; just an installer you have to download.  Next step is to set up the login profile and the assumption of the role in my sandbox account. Terraform is written and pretty much ready to go in ~/code/fargate_first.

Thereafter, need a Fargate service definition with a basic http server, and then think about next steps.

h2. Side note about permissions

When your task's containers or your EC2 instances need to access AWS services:
* Fargate tasks: create a role with the correct permissions, then list that as the `Task Definition`'s `Task Role`.
* EC2 instances: 
  * create a role with the correct permissions
  * Wrap the role in an instance profile
  * Create the EC2 instance, specifying the instance profile
  * I assume the credentials for the instance profile are provided in the instance's environment, so you can just use the AWS CLI & API directly. Not checked that. 
  * Info [here:https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile]
