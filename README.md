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

PART 1: PERMISSIONS

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

STEP 4:Create a new Application Load Balancer
