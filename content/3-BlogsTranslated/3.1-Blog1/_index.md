---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Containers and infrastructure as code, like peanut butter and jelly

> by Clare Liguori ┃ on 18 OCT 2019 ┃ in [Amazon EC2 Container Registry](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-registry/), [Amazon EC2 Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-ec2-container-service/), [Amazon Elastic Container Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-elastic-container-service/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/containers/category/compute/amazon-kubernetes-service/), [AWS Cloud Development Kit](https://aws.amazon.com/blogs/containers/category/developer-tools/aws-cloud-development-kit/), [AWS CloudFormation](https://aws.amazon.com/blogs/containers/category/management-tools/aws-cloudformation/), [AWS Fargate](https://aws.amazon.com/blogs/containers/category/compute/aws-fargate/), [Containers](https://aws.amazon.com/blogs/containers/category/containers/)

Infrastructure as code tools like AWS CloudFormation and HashiCorp Terraform enable teams to describe and automate provisioning of cloud infrastructure resources, including container-related resources like Amazon ECS services and Amazon EKS clusters. In this post, I cover why I believe infrastructure as code is especially important for containerized applications, how we use infrastructure as code with containers at Amazon, where I think infrastructure as code tooling is going, and the tools we’re building for Amazon ECS to help you get there.

## Infrastructure as code and containers: better together

Infrastructure as code and containers are better together (like peanut butter and jelly!) when developing applications deployed to the cloud. At first glance, a container image appears to be a fully self-contained application: it has all of the code and software dependencies required to run my application both locally on my laptop and in the cloud. I can share the image with others so they can also easily run my application. However, once I deploy and operate my image in the cloud, I now need a lot more configuration to scale it out, make it reliable, and make it observable. I might need to configure a container orchestrator like ECS or Kubernetes to keep a certain number of copies of my image running, and I might need other infrastructure and resources like a load balancer, DNS entries, TLS certificates, dashboards, alarms, and logging. My containerized application in the cloud might look something like this diagram, where my container image is only part of the full application:

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/complex_cloud_infrastructure.png)

Once deployed to the cloud, the container image on its own suddenly doesn’t describe the full application. Sharing my image with others doesn’t mean that someone else can easily run my application in the cloud, since they would need to re-create all this infrastructure around the image. The complete application is really best described with a combination of the container image and an infrastructure as code template containing all this configuration. Some container application “package managers” like Helm have become popular because they help you share containerized applications with others by packaging together both the image and the template.

Infrastructure as code is important not only for sharing containerized applications, but also for releasing them. The original promise of containers was that the container image bundles everything it needs to run your application, so it should behave the same way in multiple environments, for example in separate “Dev” and “Prod” environments. The image bundles all of my application’s dependencies like system libraries, so it should run exactly the same in my Dev environment and in my Prod environment, right? No more guessing and hoping that the application will behave the same once it’s deployed to a different set of instances in Prod! But, once I add in all the infrastructure shown above that also makes up my containerized application in the cloud, suddenly it is not sufficient to have the same images running in various environments: it’s now also important to have the exact same infrastructure configuration across those environments as well in order for the application to behave the same.

Slight differences in infrastructure across environments can have a significant impact on containerized application behavior and reliability. The image below shows an example where I have a different health check path configured on the load balancer in Dev vs Prod, an easy thing to overlook if I manually configure the load balancers. When I deploy my image to the Dev environment, the application behaves correctly and my end-to-end tests succeed there. But when I deploy the same image to Prod, the load balancer may send traffic to unhealthy containers because it uses the wrong health check path, causing errors in my application. I can avoid this problem by using infrastructure as code. Infrastructure as code makes infrastructure changes repeatable and predictable across multiple stages in the release process, like in Dev and Prod. It also helps replicate a staging environment as closely as possible to the production environment, so I can confidently deploy both infrastructure changes and code changes to Prod after I test them in the Dev environment.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/diff_between_dev_and_prod.png)

## Infrastructure as code with containers at Amazon

Automating infrastructure provisioning for microservices has been important for developer productivity for us internally at Amazon. Both microservices architectures and infrastructure as code are key best practices for what we at AWS call “[modern applications](https://aws.amazon.com/modern-apps/),” and we adopted both at Amazon over time. Where we once had a massive monolith that could be stood up and configured manually, automating provisioning became important once we fully adopted a microservices architecture. Standing up new infrastructure manually for tens or hundreds or thousands of microservices in a modern application became more complex, more error-prone, and took more time. Automating configuration and provisioning via templates meant that our developers could spend more time working on new features, instead of doing manual configuration. Even better, when we started deploying infrastructure as code templates through CI/CD release pipelines, developers no longer had to manually run template deployments; all they have to do now is “git push.” Amazon CTO Werner Vogels tells more of the story of our journey adopting modern application best practices at Amazon on his blog.

For containerized applications internally, our best practice is to deploy both code changes and microservice infrastructure changes through the same infrastructure as code template and in the same CI/CD release pipeline. A simplified example of our release process is shown below. The infrastructure as code template contains both the container-related configuration (for example, an ECS task definition and an ECS service) and the microservice’s infrastructure (for example, a load balancer). In the “build” stage of the pipeline, the container image is built and pushed, and the unique ID for the new container image is inserted into the infrastructure as code template. Each stage of the pipeline like “Dev” and “Prod” then deploys the same infrastructure as code template. This practice gives us confidence that deployments of the entire application are repeatable and testable, and we have full visibility in the pipeline into both the exact application code and the exact infrastructure code that is currently deployed in the production environment.

| Communication scope                       | Technologies / patterns to consider                                                        |
| ----------------------------------------- | ------------------------------------------------------------------------------------------ |
| Within a single microservice              | Amazon Simple Queue Service (Amazon SQS), AWS Step Functions                               |
| Between microservices in a single service | AWS CloudFormation cross-stack references, Amazon Simple Notification Service (Amazon SNS) |
| Between services                          | Amazon EventBridge, AWS Cloud Map, Amazon API Gateway                                      |

---

## The Pub/Sub Hub

Using a **hub-and-spoke** architecture (or message broker) works well with a small number of tightly related microservices.  
- Each microservice depends only on the *hub*  
- Inter-microservice connections are limited to the contents of the published message  
- Reduces the number of synchronous calls since pub/sub is a one-way asynchronous *push*

Drawback: **coordination and monitoring** are needed to avoid microservices processing the wrong message.

---

## Core Microservice

Provides foundational data and communication layer, including:  
- **Amazon S3** bucket for data  
- **Amazon DynamoDB** for data catalog  
- **AWS Lambda** to write messages into the data lake and catalog  
- **Amazon SNS** topic as the *hub*  
- **Amazon S3** bucket for artifacts such as Lambda code

> Only allow indirect write access to the data lake through a Lambda function → ensures consistency.

---

## Front Door Microservice

- Provides an API Gateway for external REST interaction  
- Authentication & authorization based on **OIDC** via **Amazon Cognito**  
- Self-managed *deduplication* mechanism using DynamoDB instead of SNS FIFO because:  
  1. SNS deduplication TTL is only 5 minutes  
  2. SNS FIFO requires SQS FIFO  
  3. Ability to proactively notify the sender that the message is a duplicate  

---

## Staging ER7 Microservice

- Lambda “trigger” subscribed to the pub/sub hub, filtering messages by attribute  
- Step Functions Express Workflow to convert ER7 → JSON  
- Two Lambdas:  
  1. Fix ER7 formatting (newline, carriage return)  
  2. Parsing logic  
- Result or error is pushed back into the pub/sub hub  

---

## New Features in the Solution

### 1. AWS CloudFormation Cross-Stack References
Example *outputs* in the core microservice:
```yaml
Outputs:
  Bucket:
    Value: !Ref Bucket
    Export:
      Name: !Sub ${AWS::StackName}-Bucket
  ArtifactBucket:
    Value: !Ref ArtifactBucket
    Export:
      Name: !Sub ${AWS::StackName}-ArtifactBucket
  Topic:
    Value: !Ref Topic
    Export:
      Name: !Sub ${AWS::StackName}-Topic
  Catalog:
    Value: !Ref Catalog
    Export:
      Name: !Sub ${AWS::StackName}-Catalog
  CatalogArn:
    Value: !GetAtt Catalog.Arn
    Export:
      Name: !Sub ${AWS::StackName}-CatalogArn
