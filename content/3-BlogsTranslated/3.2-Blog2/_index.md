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

Log in to the [AWS Management Console](https://aws.amazon.com/console/) and search for [AWS Cloud9](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloud9%2Fhome%3Fca-oauth-flow-id%3D9abf%26hashArgs%3D%2523%26isauthcode%3Dtrue%26oauthStart%3D1775422352359%26region%3Dus-east-1%26state%3DhashArgsFromTB_us-east-1_1cdedad3343cbf4c&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcloud9classicide&forceMobileApp=0&code_challenge=oD_UERZlGRKZgpXNAHXL-71KCoxmqs0aktGEpYTVDag&code_challenge_method=SHA-256) services in the search bar. Select AWS Cloud9 and create an AWS Cloud9 environment in the `us-east-1` region based on Amazon Linux 2.