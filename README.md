# Architecting 3 Tier Architecture on AWS
![image](https://user-images.githubusercontent.com/115881685/208869435-dd469beb-127a-4173-b39f-235a24e0a58a.png)

In this project we will create a 3 tier architecture. It sounds intimidating but I will walk you through it step by step. The first tier of our architecture is a Web tier. It will consist of 2 public subnets in separate Availability zones, and an auto scaling group of EC2 instances launching a webpage with access to the internet. The second tier is an Application tier. This tier will consist of 2 private subnets, an ASG with EC2 instances that have inbound access from the web tier. The third tier is a Database tier. This tier will have an RDS database in 2 private subnets with inbound access from the application tier above it. So let’s get to it!
## Prerequisites
AWS account

Access to command line, I will be using Windows powershell

### Step 1

First I will create a VPC. To do this navigate to VPC > Your VPCs > Create VPC. Now here we have a really fun tool where we can create a VPC with subnets and everything else we will need to create the bones of our 3 tier architecture. Click on VPC, subnets, etc. Then you can name your VPC and have it auto generate name tags for all of the resources associated with it.

![image](https://user-images.githubusercontent.com/115881685/208872346-c643cfbf-73a6-4643-a33b-e819cb13c61c.png)

Scroll down and you can choose which resources you want created with the VPC. I chose 2 availability zones with 2 public subnets and 4 private subnets. I chose to have 1 NAT gateway per availability zone to provide access to the private subnets. Be aware that there is a charge for NAT Gateways. For the VPC endpoint I chose none. Then click on Create VPC!

![image](https://user-images.githubusercontent.com/115881685/208872729-ede3fdb7-2290-4382-8aaf-1b7449b3f15d.png)

It will take a few minutes to create the VPC workflow. When it is finished it will look like the image below.

![image](https://user-images.githubusercontent.com/115881685/208872863-bfc2dcae-b129-4295-8833-8c6b07cbea29.png)

Go to your Subnets to check that you have 6 subnets total; 4 private and 2 public. In order to do the next step of the project you will need to click on the public subnets you created and change the settings of “Auto-assign public IPv4 address”. To change this setting click on Actions > Edit subnet settings > Enable auto-assign public IPv4 address and then hit Save. Do this for both of your public subnets.

![image](https://user-images.githubusercontent.com/115881685/208873006-a335e027-2f2e-43b7-9df0-7fe305733031.png)

#### Step 2













