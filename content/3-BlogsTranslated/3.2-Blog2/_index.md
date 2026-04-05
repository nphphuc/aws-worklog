---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Build and deploy a Spring Boot application to AWS App Runner with a CI/CD pipeline using Terraform

> by Irshad Buchh and Yang Xiao ┃ on 05 NOV 2021 ┃ in [Amazon EC2 Container Registry](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-registry/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/), [Technical How-to](https://aws.amazon.com/blogs/containers/category/post-types/technical-how-to/)

## Introduction

[Spring Boot](https://spring.io/projects/spring-boot) is a leading open-source framework for building Java-based web applications. It is designed to get you up and running as quickly as possible, with minimal configuration. Its opinionated take on production-ready applications makes implementing modern best practices intuitive and easy.

[AWS App Runner](https://aws.amazon.com/apprunner/) is a fully managed container application service that makes it easy for customers without any prior containers or infrastructure experience to build, deploy, and run containerized web applications and APIs. Customers simply provide source code or a container image, and App Runner automatically builds and deploys the web application, load-balances traffic, and scales on-demand.

In this blog post, we will show how to run a Spring Boot application at scale on AWS App Runner and set up a pipeline for automatic build and deployment. The sample Spring Boot application used in this blog post is the Spring PetClinic application. The PetClinic application is chosen to demonstrate the use of Spring Boot, Spring MVC, and Spring Data in building a simple but powerful database-oriented application. The PetClinic application persists its data in [Amazon Relational Database Service (Amazon RDS)](https://aws.amazon.com/rds/). The infrastructure is provisioned and managed with [Terraform](https://developer.hashicorp.com/terraform).

## Architecture

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner1.png)

## Prerequisites

Before you build the whole infrastructure, including your CI/CD pipeline, you will need to meet the following prerequisites.

## 1. AWS account

Ensure you have access to an AWS account and a set of credentials with administrator permissions. Note: In a production environment, we would recommend locking permissions down to the bare minimum needed to operate the pipeline.

## 2. Create an AWS Cloud9 environment

Log in to the [AWS Management Console](https://aws.amazon.com/console/) and search for [AWS Cloud9](https://console.aws.amazon.com/cloud9/home?region=us-east-1) services in the search bar. Select AWS Cloud9 and create an AWS Cloud9 environment in the `us-east-1` region based on Amazon Linux 2.

## Solution

## Configure the AWS Cloud9 environment

Launch the AWS Cloud9 IDE. Close the `Welcome` tab and open a new `Terminal` tab.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner2.png)

## Create and attach an IAM role for your AWS Cloud9 instance

By default, AWS Cloud9 manages temporary IAM credentials for you. Unfortunately, these are incompatible with Terraform. To get around this, you need to disable AWS Cloud9 temporary credentials, and create and attach an IAM role for your AWS Cloud9 instance.

  1. Follow [this deep link to create an IAM role with Administrator access.](https://console.aws.amazon.com/iam/home#/roles$new?step=review&commonUseCase=EC2%2BEC2&selectedUseCase=EC2&policies=arn:aws:iam::aws:policy%2FAdministratorAccess)
  2. Confirm that AWS service and EC2 are selected, then choose **Next** to view permissions.
  3. Confirm that AdministratorAccess is checked, then choose **Next: Tags** to assign tags.
  4. Take the defaults, and choose **Next: Review** to review.
  5. Enter workshop-admin for the Name, and choose **Create role**.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner3.png)

  6. Follow [this deep link to find your AWS Cloud9 EC2 instance.](https://console.aws.amazon.com/ec2/v2/home#Instances:tag:Name=aws-cloud9-;sort=desc:launchTime)
  7. Select the instance, then choose **Actions / Instance settings / Modify IAM role**. Note: If you cannot find this menu option, look under **Actions / Security / Modify IAM role**

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner4.png)

  8. Choose workshop-admin from the IAM role drop-down, and select **Apply**.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner5.png)

  9. Return to your workspace and click the gear icon (in top right corner), or click to open a new tab and choose **Open preferences**.
  10. Select **AWS settings**.
  11. Turn off AWS managed temporary credentials.
  12. Close the Preferences tab.

  ![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2021/09/22/springbootapprunner6.png)

  13. In the AWS Cloud9 terminal pane, execute the command:

  ```
  rm -vf ${HOME}/.aws/credentials
  ```

  14. As a final check, use the [GetCallerIdentity](https://docs.aws.amazon.com/cli/latest/reference/sts/get-caller-identity.html) CLI command to validate that the AWS Cloud9 IDE is using the correct IAM role.

  ```
  aws sts get-caller-identity --query Arn | grep workshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
  ```

  ## Upgrade awscli
