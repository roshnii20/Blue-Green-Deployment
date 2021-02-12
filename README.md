# Blue-Green-Deployment on AWS


Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live, and Green is idle.
As you prepare a new version of your software, deployment and the final stage of testing takes place in the environment that is not live: in this example, Green. 
Once you have deployed and fully tested the software in Green, you switch the router, so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to app deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new version on Green, you can immediately roll back to the last version by switching back to Blue.

![Intro](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/Capture.PNG)



Blue/Green is a deployment strategy that has been around forever and can be implemented using a variety of methods, CI-CD being one of the popular ones. Other methods include Chef, Ansible.

So basically, CI-CD is a tool to implement strategy of Blue-Green Deployment.

In this excercise, we will deploy application in Blue-Green environment on the AWS platform using services and tools like CodeDeploy for deployment, GitHub for storing application source code, and Jenkins for creating a build of source code, and triggering deployment on CodeDeploy.

What you will need in this lab:
•	AWS account

•	Virtual Machine

•	GitHub 

•	Jenkins

**PART 1: PERMISSIONS***

STEP 1: Create a new user with permissions to fully access EC2, AutoScaling. CodeDeploy, and S3.

STEP 2: Create S3 bucket with versioning enabled

In CodeDeploy, identity-based policies are used to manage permissions to the various resources related to the deployment process. For doing so:

STEP 3: Navigate to IAM and create a new policy in JSON name as CodeDeployPerm and paste the following: Replace your s3 bucket name in resource 

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::<your-bucket-name>"
        }
    ]
}


STEP 4: Create a new role (select EC2 service) with following policies attached

![Bg-1](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-1.png)

STEP 5: Create another role for CodeDeploy

![Bg-2](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-2.png)

PART 2: EC2 INSTANCES

STEP 1: Create a new EC2 instance and for IAM role select the EC2 role we created earlier

STEP 2: Put the following in user data while creating EC2: *(make sure to change your region if you are not in us-east-2)*

#!/bin/bash
yum update -y
yum install ruby -y
yum install aws-cli -y
yum install -y httpd 
cd /home/ec2-user/
aws s3 cp s3://aws-codedeploy-us-east-2/latest/install . --region us-east-2
chmod +x ./install
./install auto

The above list of commands will deploy all the dependencies inluding CodeDeploy agent needed by the instance to be used in CodeDeploy deployments.

STEP 3: Create a new security group with following rules:

![Bg-3](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-3.png)

We are alowing port 8080 because Jenkins uses that port to communicate.

STEP 4: Create a new Application Load Balancer

We are using Application Load Balancer(ALB) so that we can configure multiple listener rules by choosing our instances as Target groups. Because of ALB, we do not need to maintain separate instances for both Blue and Green environment.


Start creating ALB and choose the security group we created earlier, register our instances to Targets, and create it.


STEP 5: Create a template from our EC2 instance *(Choose VPC for Networking Platform and select the Security Group we have created)


Now that our EC2 instances has the required dependencies in it, ALB attached and we want the same configuration for all our instances so are creating a template from our instance so that everything our instance has, will be present in the template and we will use this template to create an Auto-Scaling Group in next step. 


STEP 6: Creating an Auto-Scaling group

Here we create an AS group using the template that we created in the previous step. Enable Load Balancing and select out Target Group

Based on the size that we of an Auto Scaling group is determined by the number of instances set in the desired capacity field, these can be set manually or by using automatic scaling. An Auto Scaling starts by launching desired number of instances, it maintains this number of instances by performing periodic health checks, if an instance becomes unhealthy, it replaces that unhealthy instance with another instance.

PART 3: AWS CodeDeploy

STEP 1: We will create a new application on AWS CodeDeploy  

Choose EC2/On premises as compute platform, CodeDeploy Service role that we created earlier.

In the Deployment type, select Blue-Green Deployment, choose the ALB we had created in Part 1, Deployment Configuration as CodeDeployDefault.OneAtATime and create the Deployment Group


PART 4: GITHUB

STEP 1: Create a new repository on GitHub, create a new folder called as scripts and write 3 files in the way I have added in this repository


STEP 2: Create an appspec.yml file and index.html file in your repo as following (Not inside the scripts folder)


PART 5: JENKINS

In this exercise, we are going to use Jenkins to create our source build and create deployment

STEP 1: Install Jenkins on your Virtual Machine (Follow official Jenkins installation guide for steps)


STEP 2: We simply need to use the IP of our Jenkins server, which in this case is 192.168.1.28 and add a port 8080 after the IP address to open the Jenkins GUI (192.168.1.28:8080)


STEP 3: Creating a new job on Jenkins (Use freestyle project)


STEP 4: For Source Code Management, choose Git and add your repository link


STEP 5: : Select a Post-Build job as Deploy application to CodeDeploy and enter all AWS CodeDeploy details as well as S3 bucket and IAM Credentials 

Successful output:

![Bg-5](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-6.png)


STEP 6: Now go back to AWS Console and check for Deployment

![Bg-4](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-4.png)


Successful output:
![Bg-5](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-5.png)



STEP 7: Now go to your Load Balancer and check whether the traffic has shifted


SUccessful output:

![Bg-7](https://github.com/roshnii20/Blue-Green-Deployment/blob/main/Pictures/BG-7.PNG)


Now you can make any changes to your index.html file and use Jenkins to trigger it.


***End***
