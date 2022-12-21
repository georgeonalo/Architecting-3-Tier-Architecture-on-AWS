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

Now we will work on our Web tier. Head over to the EC2 Dashboard > Auto Scaling Groups > Create Auto Scaling group. Here you will name your Auto Scaling group for the public EC2 instances we will create. Then click on Create a launch template.

![image](https://user-images.githubusercontent.com/115881685/208874304-3efbf445-907f-4a36-8731-29e98da6e33b.png)

Here you will name your public launch template, choose the AMI and instance type. I chose Amazon Linux and t2.micro. Specify your key pair. Then under Network Settings we will create a security group to allow HTTP and SSH access to our instances in the public subnets.

![image](https://user-images.githubusercontent.com/115881685/208874425-6a7cd5d8-1458-49c7-bc6c-0c979186fdb8.png)

![image](https://user-images.githubusercontent.com/115881685/208874550-d84e6338-799e-455b-b3f7-5b132bddd53d.png)

Scroll down and under Advanced Details I will add a bootstrap to the User Data textbox. The bootstrap will install an Apache webserver on our instances and provision a webpage with a script. See bootstrap below:

#!/bin/bash

yum update -y

systemctl start httpd

systemctl enable httpd

echo "<html><body><h1></h1></body></html>" > /var/www/html/index.html

Click create launch template. Go back to Auto Scaling Groups and your new launch template will populate as an option. Next you will choose launch options. Select the VPC you created and select the 2 public subnets.

![image](https://user-images.githubusercontent.com/115881685/208874968-591be1e0-5968-4d26-a83e-14c741121c25.png)

Click Next, leave this page default and hit Next. On the Configure group size and scaling policies page; I will input 2 desired instances, 2 minimum and 5 maximum.

![image](https://user-images.githubusercontent.com/115881685/208875143-ce84a27d-c8c5-495b-8276-f59040459470.png)


Then hit next through to Review and then click create Auto Scaling group. If we go over to our instances we should have 2 running in 2 Availability zones.

![image](https://user-images.githubusercontent.com/115881685/208875276-299accee-5843-4f60-aaf7-02c4db4108cc.png)


To check that the instance is running with a webpage, grab the IPv4 address and paste it in your browser.

![image](https://user-images.githubusercontent.com/115881685/208875441-08ca24a9-09f8-45df-86aa-0e87ac72835f.png)




















