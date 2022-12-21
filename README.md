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

echo"<html><body><h1></h1></body></html>" > /var/www/html/index.html

Click create launch template. Go back to Auto Scaling Groups and your new launch template will populate as an option. Next you will choose launch options. Select the VPC you created and select the 2 public subnets.

![image](https://user-images.githubusercontent.com/115881685/208874968-591be1e0-5968-4d26-a83e-14c741121c25.png)

Click Next, leave this page default and hit Next. On the Configure group size and scaling policies page; I will input 2 desired instances, 2 minimum and 5 maximum.

![image](https://user-images.githubusercontent.com/115881685/208875143-ce84a27d-c8c5-495b-8276-f59040459470.png)


Then hit next through to Review and then click create Auto Scaling group. If we go over to our instances we should have 2 running in 2 Availability zones.

![image](https://user-images.githubusercontent.com/115881685/208875276-299accee-5843-4f60-aaf7-02c4db4108cc.png)


To check that the instance is running with a webpage, grab the IPv4 address and paste it in your browser.

![image](https://user-images.githubusercontent.com/115881685/208875441-08ca24a9-09f8-45df-86aa-0e87ac72835f.png)

And it works! Thus completing the Web tier of our architecture.


###### Step 3

Now on to the Application tier. In this section we will put EC2 instances, via a Auto Scaling group, into 2 private subnets. Note that this is not a true application tier as we don’t have any provided code to run on the EC2 instances.

Navigate to Create launch template. I will use the same AMI and instance type as the previous launch template. Associate a key pair and then we will create a security group. This time since these are private subnets we will want to only allow access from our web tier security group and to SSH. As shown below.

![image](https://user-images.githubusercontent.com/115881685/208896861-a00e8163-da80-4fd0-8c67-b076f5f90786.png)


![image](https://user-images.githubusercontent.com/115881685/208896912-7af838ba-c26e-47ce-8f99-10935b3c38f7.png)

We can now create the launch template. Then in ASG choose our new template we just created. Click Next and choose launch options. Choose the correct VPC and this time choose 2 private subnets to launch our instances in to.

![image](https://user-images.githubusercontent.com/115881685/208897026-19ca37a0-73b8-4fc1-ae75-bb6bc042c34c.png)


Use the same options as the web tier ASG and create your auto scaling group. You will now have 2 Auto Scaling groups.

![image](https://user-images.githubusercontent.com/115881685/208897127-2b8262bc-cbcd-4bff-a6bd-619c6c675bf9.png)

And 2 new instances!

![image](https://user-images.githubusercontent.com/115881685/208897184-9eb6eed1-ab07-4cc5-a0c7-dba1ef4b56db.png)


To verify if we have access to the Private subnets from the public subnets we will attempt to ping a private subnet from the command line. You can do this by grabbing the public IPv4 of one of your public instances and SSH into that instance. I already have my key pair attached, you may need to add your key pair when you SSH.

![image](https://user-images.githubusercontent.com/115881685/208897311-5320f39f-340f-4619-b24c-f28fc1d57103.png)


We will then ping the Private IP address, by using ping .

![image](https://user-images.githubusercontent.com/115881685/208897375-c492bad2-8522-4f34-a15c-8e9339607e84.png)

It returned an amount which shows it was successful. I will now see if I can connect to my private instance using SSH forwarding agent. You can do this by adding -A when you ssh into a public instance. SSH into your public IPv4 address using the following command.

ssh -A ec2-user@107.21.69.189

Then SSH into the private instance from the public instance by using the private IPv4 address.

![image](https://user-images.githubusercontent.com/115881685/208897618-c1f2b72e-ccb8-4824-a63e-9ee2cad562cc.png)


It worked, we are now in the private instance! That concludes out Application tier.

###### Step 4

























