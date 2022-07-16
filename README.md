# TOPIC: Rollback-Project

![image](https://user-images.githubusercontent.com/40290711/179347404-08fc5cc0-e715-400e-a27e-1ea110d15546.png)


## Overview
> In this project, you will write a reusable command that rolls back changes. The current project is in continuation to the previous Project: Config and Deployment (InfrastructureCD in my repro).

## Rollback Deployment
> Rollback Deployment means, going back to the previous instance of the deployment if there is some issue with the current deployment.

## Prerequisites
> Key pair - You should have an AWS EC2 key pair. We are assuming the key pair name is value12.pem.

> EC2 Ubuntu instance - You should have an EC2 Ubuntu instance running in your AWS account.

> CircleCI project - A public Github repository set up in the [CircleCI](https://app.circleci.com/). AWS Access keys and EC2 key pair (SSH key) must have saved in the CircleCI project settings.

> You should have finished the previous:

> - Exercise: [Remote Control Using Ansible](https://github.com/OloruntobiOlurombi/Remote-Control-Using-Ansible-2.git),
> - Exercise: [Infrastructure Creation](https://github.com/OloruntobiOlurombi/Infrastructure-Creation.git), and
> - Exercise: [InfrastructureCD](https://github.com/OloruntobiOlurombi/InfrastructureCD.git),
> - And have the following files in your repo and local:

```
└── template.yml
└── .circleci
    └─── config.yml
```
> - Ensure to have CloudFormation template named template.yml that will create some infrastructure. This should be checked into your git repo. You can use your own from previous lessons or you can try this example:

```
# Exercise - Rollback
AWSTemplateFormatVersion: 2010-09-09
Description: CICD - Author - Oloruntobi - Rollback Job 
Resources:
  EC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      # Change this, as applicable to you      
      KeyName: value12
      # Change this, as applicable to you
      # You may need to find out what instance types are available in your region - use https://cloud-images.ubuntu.com/locator/ec2/
      ImageId: 'ami-052efd3df9dad4825' 
      InstanceType: t2.micro
      Tags: 
        - Key: "Project"
          Value: "udacity-12"

  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0 
```    
> Ensure that the KeyName and ImageID are correct.

## Steps 

> In this exercise, you will work in the "commands" section of the CircleCI config file. There may be many places where we want to rollback your changes. That means we need a "command" so that we can reuse the code.

### Step 1

> Add a command to your Circle CI config file called ***destroy_environments***. This command should use [cloudformation](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/delete-stack.html) delete-stack to delete the stack that the job previously created. It wil look like:

```  
commands:
  # Exercise - Rollback 
  destroy_environment: 
    steps:
      - run: 
          name: Destroy environment 
          # ${CIRCLE_WORKFLOW_ID} is a Built-in environment variable 
          # ${CIRCLE_WORKFLOW_ID:0:5} takes the first 5 chars of the variable CIRCLE_CI_WORKFLOW_ID
          when: on_fail 
          command: |
            aws cloudformation delete-stack --stack-name myStack-${CIRCLE_WORKFLOW_ID:0:7}

``` 

### Step 2

> Write a ***create_infrastructure job*** that creates a new stack using the template you just created.

``` 
 create_infrastructure:
    docker:
      - image: amazon/aws-cli 
    steps: 
      - checkout 
      - run:
          name: Create Cloudformation Stack 
          command: |
            aws cloudformation deploy \
             --template-file template.yml \
             --stack-name myStack-${CIRCLE_WORKFLOW_ID:0:7} \
             --region us-east-1  
      - run: return 1
      - destroy_environment 
``` 

> Add the job above in the workflows section, and comment out the previous jobs. It will save you some build time.

> After your pipeline executes successfully, you should be able to verify the stack was created and then deleted. In the end, you should have no additional stacks in your AWS CloudFormation console.

### THE END
