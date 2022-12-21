# Architecting 3 Tier Architecture on AWS
![image](https://user-images.githubusercontent.com/115881685/208869435-dd469beb-127a-4173-b39f-235a24e0a58a.png)
In this project we will create a 3 tier architecture. It sounds intimidating but I will walk you through it step by step. The first tier of our architecture is a Web tier. It will consist of 2 public subnets in separate Availability zones, and an auto scaling group of EC2 instances launching a webpage with access to the internet. The second tier is an Application tier. This tier will consist of 2 private subnets, an ASG with EC2 instances that have inbound access from the web tier. The third tier is a Database tier. This tier will have an RDS database in 2 private subnets with inbound access from the application tier above it. So let’s get to it!
#### Prerequisites
AWS account
Access to command line, I will be using Windows powershell
####### Step 1
First I will create a VPC. To do this navigate to VPC > Your VPCs > Create VPC. Now here we have a really fun tool where we can create a VPC with subnets and everything else we will need to create the bones of our 3 tier architecture. Click on VPC, subnets, etc. Then you can name your VPC and have it auto generate name tags for all of the resources associated with it.

My first repository on GitHub
