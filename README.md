# TOPIC: Rollback-Project

## Overview
> In this project, you will write a reusable command that rolls back changes. The current project is in continuation to the previous Project: Config and Deployment (InfrastructureCD in my repro).

## Rollback Deployment
> Rollback Deployment means, going back to the previous instance of the deployment if there is some issue with the current deployment.

## Prerequisites
> Key pair - You should have an AWS EC2 key pair. We are assuming the key pair name is value12.pem.

> EC2 Ubuntu instance - You should have an EC2 Ubuntu instance running in your AWS account.

> CircleCI project - A public Github repository set up in the [CircleCI](https://app.circleci.com/). AWS Access keys and EC2 key pair (SSH key) must have saved in the CircleCI project settings.

> You should have finished the previous:

> - Exercise: Remote Control Using Ansible,
> - Exercise: Infrastructure Creation, and
> - Exercise: InfrastructureCD,
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
