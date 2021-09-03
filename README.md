# DevOps-task

* Step 1: Install and configure the jenkins
* Step 2: create the pipeline for triggering the git.
* Step 3: install the npm on the ec2-instance and create a own ami of it.
* Step 4: Create a cloudformation template using my ami.
* Step 5. Setup the auto scaling part.


## Step 1:  Install and configure the jenkins

1. Launch an EC2-instance (Using RHEL AMI) and connect with through any SSH remote software like MobaXterm and putty etc.
![alt text for screen readers](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(1).png)

* **Note** : bash script for installing jenkins server.
```
#bin/bash
sudo yum install wget -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum -y install  java-11-openjdk java-11-openjdk-devel
sudo yum install jenkins --nobest -y
sudo systemctl daemon-reload
sudo systemctl start jenkins
sudo systemctl  enable jenkins
```


2. Download the jenkins script which I created for intsalling the jenkins and run this script using ``bash main.sh``.
![alt text for screen readers](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(2).png)

3. Now check whether jenkins server is running fine or not using ``systemctl status jenkins`` command.
![jenkins status](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(3).png)

4. Now our jenkins server is ready for use so go the brower and type ``http://instance_public_ip:8080/``.
![login with jenkins](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(4).png)

5. Now go inside ``/var/lib/jenkins/secrets/initial/AdminPassward`` and retrive the default passward of jenkins server which we will use to login to Jenkins server initially.
![jenkins default passward](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(5).png)

6. Copy that passward and paste in the jenkins login page and after that create your new username and passward.
![create the username n passward](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(8).png)

7. Now jenkins server is ready.
![jenkins is ready](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(9).png)

## Step 2: create the pipeline for triggering the git.


1. Click on ``Manage jenkins >> manage plugins`` and install ``Github Integration`` for triggering the changes are pushed to git.
![install github integration plugin](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(15).png)

2. Create the create a repo where you will save the stuff related to this task and go inside the ``sitting >> webhooks`` of this repo for creating the triggers.
![webhook](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(23).png)

3. Create the pipeline that triggers when changes are pushed to git so click on ``new item`` and choose ``Github hook triggr for GITScm polling`` then save it.
![create the jib](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(21).png)

4. Whenever we will push any change in this repo as the following one, jenkins server will trigger that.
![triggering the changes](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(25).png)

5. Now you can see in the output.
![output](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(28).png)

## Step 3: Install the npm on the ec2-instance and then create a own ami of it.

Here I will create a **myami with npm node** that I will use in the cloudformation.

1. So launch an EC2-instance for installing **nodejs** and now install the latest version of Node.js, you need to install development tools such as make, git, gcc on your system using ``dnf groupinstall "Development Tools"`` command.
![developments tools](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(29).png)

2.  Next, check the available Node.js package contained in the Application Stream Repository using ``dnf module list nodejs`` command.
![module list](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(30).png)

3. Install the default Node.js module by running the ``dnf module install nodejs`` command.
![install nodejs](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(31).png)

4. If you are a developer, you can use the development profile to install the libraries that enable you to build dynamically loadable modules, as follows:

```
dnf module install nodejs/development
```
5. To install a minimal set of Node.js packages, run the following command.

```
dnf module install nodejs/minimal
```
6. Once you have installed Node.js on your system, use the ``npm --version`` commands to check the version of the **nodejs**.
![npm --version](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(36).png)

7. As you have seen that **npm** has been installed in the our RHEL instance, now we can create our **myami**, so select your npm instance and click on ``actions >> images and templates >> create image``.
![myami](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(37).png)

8. Give the name to image according to you.
![name convation](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(32).png)

9. Now **myami** is ready to use.
![myimage](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(38).png)

##  Step 4: Create a cloudformation template using myami.
In this section, we will create the cloudformation template so that we can use that as we need.

1. Create a **json template** for cloudformation template.
```
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "Template to Create an EC2 instance in a VPC",
    "Parameters": {
        "ImageId": {
            "Type": "String",
            "Description": "RHEL Linux  AMI for Europe (Frankfurt)eu-central-1",
            "Default": "ami-0ccf20ad3aecd35df"
        },
        "VpcId": {
            "Type": "String",
            "Description": "VPC id",
            "Default": "vpc-5807b232"
        },
        "SubnetId": {
            "Type": "String",
            "Description": "Subnet in which to launch an EC2",
            "Default": "subnet-9d58c4f7"
        },
        "AvailabilityZone": {
            "Type": "String",
            "Description": "Availability Zone into which instance will launch",
            "Default": "eu-central-1a"
        },
        "InstanceType": {
            "Type": "String",
            "Description": "Choosing  t2 micro because it is free",
            "Default": "t2.micro"
        },
        "KeyName": {
            "Description": "SSH Keypair to login to the instance",
            "Type": "AWS::EC2::KeyPair::KeyName",
            "Default": "santosh"
        }
    },
    "Resources": {
        "DemoInstance": {
            "Type": "AWS::EC2::Instance",
            "Properties": {
                "ImageId": {
                    "Ref": "ImageId"
                },
                "InstanceType": {
                    "Ref": "InstanceType"
                },
                "AvailabilityZone": {
                    "Ref": "AvailabilityZone"
                },
                "KeyName": {
                    "Ref": "KeyName"
                },
                "SecurityGroupIds": [
                    {
                        "Ref": "DemoSecurityGroup"
                    }
                ],
                "SubnetId": {
                    "Ref": "SubnetId"
                }
            }
        },
        "DemoSecurityGroup": {
            "Type": "AWS::EC2::SecurityGroup",
            "Properties": {
                "VpcId": {
                    "Ref": "VpcId"
                },
                "GroupDescription": "SG to allow SSH access via port 22",
                "SecurityGroupIngress": [
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "22",
                        "ToPort": "22",
                        "CidrIp": "0.0.0.0/0"
                    }
                ],
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "SSH-SG"
                    }
                ]
            }
        }
    },
    "Outputs": {
        "DemoInstanceId": {
            "Description": "Instance Id",
            "Value": {
                "Ref": "DemoInstance"
            }
        }
    }
}
```
2. After creating this template file, go your AWS console and search **cloudformation**, then click on **create stack** and following below one.
![create stack](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(33).png)

3. As you can see the graphcal representaion as well of this template.
![create stack](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(34).png)

4. Give the stack name according to you.
![create stack](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(35).png)

5. Review this stack and then save it, after sometime, you wil see something like this.
![cloudformation](https://github.com/santosh10386/devOps-task/blob/main/Screenshots/Screenshot%20(39).png)



