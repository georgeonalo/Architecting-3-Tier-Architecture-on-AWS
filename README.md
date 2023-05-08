# Architecting 3 Tier Architecture on AWS
![image](https://user-images.githubusercontent.com/115881685/208869435-dd469beb-127a-4173-b39f-235a24e0a58a.png)

In this project i created a 3 tier architecture. The first tier of our architecture is a Web tier. It consist of 2 public subnets in separate Availability zones, and an auto scaling group of EC2 instances launching a webpage with access to the internet. The second tier is an Application tier. This tier consist of 2 private subnets, an ASG with EC2 instances that have inbound access from the web tier. The third tier is a Database tier. This tier will have an RDS database in 2 private subnets with inbound access from the application tier above it. 

## Prerequisites
* AWS account

* Access to command line, I will be using Windows powershell

## Step 1
First I will create a VPC. To do this navigate to VPC > Your VPCs > Create VPC. Now here we have a really fun tool where we can create a VPC with subnets and everything else we will need to create the bones of our 3 tier architecture. Click on VPC, and more. Then you can name your VPC and have it auto generate name tags for all of the resources associated with it.

![image](https://user-images.githubusercontent.com/115881685/218316845-7f797df7-b607-4330-9b4c-6755e373cda0.png)

Scroll down and you can choose which resources you want created with the VPC. I chose 2 availability zones with 2 public subnets and 4 private subnets. I chose to have 1 NAT gateway per availability zone to provide access to the private subnets. Be aware that there is a charge for NAT Gateways. For the VPC endpoint I chose none. Then click on Create VPC!

![image](https://user-images.githubusercontent.com/115881685/218316901-41d0a8d6-3a2d-4a5c-937c-731414e5ad44.png)
![image](https://user-images.githubusercontent.com/115881685/218316927-fa777fd6-6b59-441a-93e1-478e1848c4e3.png)



It will take a few minutes to create the VPC workflow. When it is finished it will look like the image below.

![image](https://user-images.githubusercontent.com/115881685/218316974-d7ef3b8e-bdf0-420f-a629-ae6adecace9f.png)
![image](https://user-images.githubusercontent.com/115881685/218317001-6c83683f-082f-464f-827f-1b4e1f8c23c3.png)


Go to your Subnets to check that you have 6 subnets total; 4 private and 2 public. In order to do the next step of the project you will need to click on the public subnets you created and change the settings of “Auto-assign public IPv4 address”. To change this setting click on Actions > Edit subnet settings > Enable auto-assign public IPv4 address and then hit Save. Do this for both of your public subnets.

![image](https://user-images.githubusercontent.com/115881685/218317064-6de89a32-59ac-4c2b-a530-3e1b68bc4a05.png)

### Step 2

Now we will work on our Web tier. Head over to the EC2 Dashboard > Auto Scaling Groups > Create Auto Scaling group. Here you will name your Auto Scaling group for the public EC2 instances we will create. Then click on Create a launch template.

![image](https://user-images.githubusercontent.com/115881685/218317148-efd82fb7-0fe3-4374-b73c-98fe8cf40438.png)

Here you will name your public launch template, choose the AMI and instance type. I chose Amazon Linux and t2.micro. Specify your key pair. Then under Network Settings we will create a security group to allow HTTP and SSH access to our instances in the public subnets.

![image](https://user-images.githubusercontent.com/115881685/218317213-a0f7bb41-a0e3-4c7a-8b8b-34d3eee4c332.png)

![image](https://user-images.githubusercontent.com/115881685/218317245-668393bc-dc82-4003-8ab1-5d10d7ec92a1.png)

Scroll down and under Advanced Details I will add a bootstrap to the User Data textbox. The bootstrap will install an Apache webserver on our instances and provision a webpage with a script. See bootstrap below:

```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<html><body><h1>Welcome to the 3 Tier Architecture</h1></body></html>" > /var/www/html/index.html
```


Click create launch template. Go back to Auto Scaling Groups and your new launch template will populate as an option. Next you will choose launch options. Select the VPC you created and select the 2 public subnets.


![image](https://user-images.githubusercontent.com/115881685/218317342-a3ec84cb-3798-4362-b584-0765139f81f6.png)

Click Next, leave this page default and hit Next. On the Configure group size and scaling policies page; I will input 2 desired instances, 2 minimum and 5 maximum.

![image](https://user-images.githubusercontent.com/115881685/218317397-2a4a2726-e33f-495a-b18f-60f6d7d9a277.png))


Then hit next through to Review and then click create Auto Scaling group. If we go over to our instances we should have 2 running in 2 Availability zones.

![image](https://user-images.githubusercontent.com/115881685/218317460-d70d5473-e8a0-48e2-9430-628630aaa0c4.png)


To check that the instance is running with a webpage, grab the IPv4 address and paste it in your browser.


![image](https://user-images.githubusercontent.com/115881685/218317486-60336bff-f34c-43d2-87f1-fcb481f6e164.png)

And it works! Thus completing the Web tier of our architecture.


## Step 3

Now on to the Application tier. In this section we will put EC2 instances, via a Auto Scaling group, into 2 private subnets. Note that this is not a true application tier as we don’t have any provided code to run on the EC2 instances.

Navigate to Create launch template. I will use the same AMI and instance type as the previous launch template. Associate a key pair and then we will create a security group. This time since these are private subnets we will want to only allow access from our web tier security group and to SSH. As shown below.

![image](https://user-images.githubusercontent.com/115881685/218317532-1ef95aad-7eda-420e-914c-129216a7403e.png)


![image](https://user-images.githubusercontent.com/115881685/218317576-f9928a66-025f-4fff-bcd7-565573175760.png)

We can now create the launch template. Then in ASG choose our new template we just created. Click Next and choose launch options. Choose the correct VPC and this time choose 2 private subnets to launch our instances in to.

![image](https://user-images.githubusercontent.com/115881685/218317616-a3fd12dd-9bad-42f6-9de2-a84248d5e1fd.png)


Use the same options as the web tier ASG and create your auto scaling group. You will now have 2 Auto Scaling groups.

![image](https://user-images.githubusercontent.com/115881685/218317669-ed369ec7-0e07-485e-bc6a-ca5090e3f662.png)

And 2 new instances!

![image](https://user-images.githubusercontent.com/115881685/218317720-825e8fae-a9df-42fd-8a56-0a91f6614bc6.png)


To verify if we have access to the Private subnets from the public subnets we will attempt to ping a private subnet from the command line. You can do this by grabbing the public IPv4 of one of your public instances and SSH into that instance. I already have my key pair attached, you may need to add your key pair when you SSH.

![image](https://user-images.githubusercontent.com/115881685/218317795-5ad2a450-3ce7-4745-8b99-a2eebad2d222.png)


We will then ping the Private IP address, by using ping .

![image](https://user-images.githubusercontent.com/115881685/218318817-03eacdee-8f45-4459-b854-3f72edc6f78c.png)

It returned an amount which shows it was successful. I will now see if I can connect to my private instance using SSH forwarding agent. You can do this by adding -A when you ssh into a public instance. SSH into your public IPv4 address using the following command.


```
ssh -A ec2-user@44.199.236.130
```


Then SSH into the private instance from the public instance by using the private IPv4 address.

![image](https://user-images.githubusercontent.com/115881685/218317926-0bbba878-721e-4c36-89f1-64e1e6862cb6.png)


It worked, we are now in the private instance! That concludes out Application tier.

## Step 4

For our 3rd and final tier we will add a database to the private subnets. In the AWS console navigate to Amazon RDS. From the dashboard click on Subnet groups > Create DB subnet group. Here we will create a new subnet group. Name it and select your VPC.


![image](https://user-images.githubusercontent.com/115881685/218317981-fe466e3e-5cf9-448a-b135-5f22cfd67550.png)

I will create a Multi-AZ DB instance to provide higher availability and data redundancy by choosing both Availability zones. Be sure to choose the private subnets you have not used yet, that do not contain EC2 instances. Then create the group.

![image](https://user-images.githubusercontent.com/115881685/218318037-e11aaad8-1077-4b4e-952c-98e71b162a01.png)


Navigate back to the RDS dashboard and click Create Database. Select Standard create and MySQL.


![image](https://user-images.githubusercontent.com/115881685/218318080-9506ba1e-ff14-4708-857c-34f031a34d0e.png)

For the template I chose Free tier. Under Settings keep them default but add a Master password for your admin. Save the password for use later. For Instance configuration and Storage keep default. Under Connectivity select your VPC and the subnet group you just created. For Public access choose “no” since this is a Private database and will only be accessible from inside the VPC. Then click to create a security group.

![image](https://user-images.githubusercontent.com/115881685/218318154-1cc6eb60-d88e-4b60-97b1-e5be36ea31f0.png)



Click Create Database! After a moment your database will be created. Click on it and scroll to Connectivity and security. We need to edit our security group to allow inbound access from the application tier. To do this click on your database security group.


![image](https://user-images.githubusercontent.com/115881685/218318229-cfe5b102-d453-4943-b122-f7572fa09b6c.png)


This will bring you to Security groups. Here you can edit the inbound rules.

![image](https://user-images.githubusercontent.com/115881685/218318289-7cd656dd-b098-45cb-9987-f6a6bccb8992.png)


Change the source to your application tier security group and hit save.

![image](https://user-images.githubusercontent.com/115881685/218318353-770d4eb2-bfab-4613-a3e4-899e453a293d.png)

Now to test that our private EC2 instances from the application tier are able to access the database we will head to the command line! First you will need to SSH into one of your public subnets and include -A to add a SSH forwarding agent. Then from the public subnet SSH into one of the private subnets with your private IPv4 address.

```
ssh -A ec2-user@10.0.142.78
```

Here you will need to have mariadb installed to access the MySQL database. This is a simple step, type the command below.


```
sudo yum install mariadb
```



Now we should be able to access our database from the private instance. We can do so with the following command.

```
mysql -h database-1.cinrc1votzp3.us-east-1.rds.amazonaws.com -P 3306 -u admin -p
```

The -h is your database endpoint which can be found in the console in your database under Connectivity and security. The -P specifies the port 3306. The user, -u, is admin. You will then be prompted to enter a password, the one you created earlier for admin.

![image](https://user-images.githubusercontent.com/115881685/218319366-cdd8dca5-a98c-4dbf-a646-d95938b993a4.png)


We successfully accessed the database tier from the application tier! All 3 tiers are up and running! Congratulate yourself on creating a 3 tier architecture. Now go back and tear it all down so as not to be charged.

































