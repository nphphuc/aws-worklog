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

Once deployed to the cloud, the container image on its own suddenly doesn’t describe the full application. Sharing my image with others doesn’t mean that someone else can easily run my application in the cloud, since they would need to re-create all this infrastructure around the image. The complete application is really best described with a combination of the container image and an infrastructure as code template containing all this configuration. Some container application “package managers” like [Helm](https://helm.sh/) have become popular because they help you share containerized applications with others by packaging together both the image and the template.

Infrastructure as code is important not only for sharing containerized applications, but also for releasing them. The original promise of containers was that the container image bundles everything it needs to run your application, so it should behave the same way in multiple environments, for example in separate “Dev” and “Prod” environments. The image bundles all of my application’s dependencies like system libraries, so it should run exactly the same in my Dev environment and in my Prod environment, right? No more guessing and hoping that the application will behave the same once it’s deployed to a different set of instances in Prod! But, once I add in all the infrastructure shown above that also makes up my containerized application in the cloud, suddenly it is not sufficient to have the same images running in various environments: it’s now also important to have the exact same infrastructure configuration across those environments as well in order for the application to behave the same.

Slight differences in infrastructure across environments can have a significant impact on containerized application behavior and reliability. The image below shows an example where I have a different health check path configured on the load balancer in Dev vs Prod, an easy thing to overlook if I manually configure the load balancers. When I deploy my image to the Dev environment, the application behaves correctly and my end-to-end tests succeed there. But when I deploy the same image to Prod, the load balancer may send traffic to unhealthy containers because it uses the wrong health check path, causing errors in my application. I can avoid this problem by using infrastructure as code. Infrastructure as code makes infrastructure changes repeatable and predictable across multiple stages in the release process, like in Dev and Prod. It also helps replicate a staging environment as closely as possible to the production environment, so I can confidently deploy both infrastructure changes and code changes to Prod after I test them in the Dev environment.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/diff_between_dev_and_prod.png)

## Infrastructure as code with containers at Amazon

Automating infrastructure provisioning for microservices has been important for developer productivity for us internally at Amazon. Both microservices architectures and infrastructure as code are key best practices for what we at AWS call “[modern applications](https://aws.amazon.com/modern-apps/),” and we adopted both at Amazon over time. Where we once had a massive monolith that could be stood up and configured manually, automating provisioning became important once we fully adopted a microservices architecture. Standing up new infrastructure manually for tens or hundreds or thousands of microservices in a modern application became more complex, more error-prone, and took more time. Automating configuration and provisioning via templates meant that our developers could spend more time working on new features, instead of doing manual configuration. Even better, when we started deploying infrastructure as code templates through CI/CD release pipelines, developers no longer had to manually run template deployments; all they have to do now is “git push.” Amazon CTO Werner Vogels tells more of the story of our journey adopting modern application best practices at Amazon [on his blog.](https://www.allthingsdistributed.com/2019/08/modern-applications-at-aws.html)

For containerized applications internally, our best practice is to deploy both code changes and microservice infrastructure changes through the same infrastructure as code template and in the same CI/CD release pipeline. A simplified example of our release process is shown below. The infrastructure as code template contains both the container-related configuration (for example, an ECS task definition and an ECS service) and the microservice’s infrastructure (for example, a load balancer). In the “build” stage of the pipeline, the container image is built and pushed, and the unique ID for the new container image is inserted into the infrastructure as code template. Each stage of the pipeline like “Dev” and “Prod” then deploys the same infrastructure as code template. This practice gives us confidence that deployments of the entire application are repeatable and testable, and we have full visibility in the pipeline into both the exact application code and the exact infrastructure code that is currently deployed in the production environment.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/Infra_as_Code_and_Containers_blog_post_diagrams.png)

## The next phase: architecture as code

I believe that we’ll increasingly see infrastructure as code tooling help developers to think beyond modeling and provisioning individual infrastructure resources, but instead to think in terms of modeling and provisioning cloud architecture. As developers, when we’re designing a cloud application, we don’t typically think in terms of infrastructure. We think in terms of architecture: we draw high-level boxes and arrows on the whiteboard to describe microservices and service-to-service interactions, not details like load balancers and security groups. The below image is an architecture diagram I drew recently for a small application I’m building to tally votes. It’s similar to many whiteboard designs: it’s high level and doesn’t drill into the infrastructure that makes up each microservice box.

![image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/10/01/whiteboard_drawing.png)

Lots of the microservice boxes in whiteboard diagrams like the image above tend to follow similar patterns of “types” or “kinds” of microservices: an API service behind a load balancer, a service that pulls work from a queue, a scheduled job that runs once a day, etc. The infrastructure for one kind of microservice, like an “API service,” tends to look the same across many API services. Usually if we compare the infrastructure as code templates for all of our API services, only a few attributes change from template to template, for example the container image and the scale of the service.

Once we discover patterns in our infrastructure that represent different kinds of services, these patterns can be captured in reusable infrastructure as code components and shared across many microservices. Then developers can describe their high-level architecture in a template: they can specify the “kind” of microservice they are building and how it relates to other microservices in their architecture, instead of specifying individual infrastructure resources over and over again in a new template for each microservice they build. When configuration changes are needed for one type of microservice, for example changing the load balancer health check path for an “API service,” the change can be made in the re-usable component and then immediately rolled out to each microservice using that pattern, without needing to make changes in every microservice’s template.

I believe we’ll start to see more “architecture as code” tooling emerge that enables developers to describe their microservices in terms of high-level infrastructure patterns. The tooling will take care of combining the infrastructure pattern components with the developer’s microservice description to automatically provision the necessary infrastructure resources, like container orchestrator configuration, load balancers, security groups, and IAM roles. Today, there are already many options for creating and distributing infrastructure patterns across microservices and across teams to practice “architecture as code” in your organization, like [CloudFormation macros](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-macros.html), [AWS Cloud Development Kit constructs](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html#constructs_author), [Terraform modules](https://developer.hashicorp.com/terraform/language/modules), and [Kubernetes custom resource definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions).

## Architecture as code with Amazon ECS and AWS CDK

In the AWS Containers team last year, we started exploring the kinds of microservices were commonly running on Amazon ECS and what those patterns looked like in terms of infrastructure resources. We wanted to enable teams to easily adopt the architecture as code model. In this model, developers would only specify what kind of containerized service they wanted to run and their Dockerfile, and all the infrastructure resources including VPCs, subnets, ECS configuration, ECR registry, load balancer, and more would be provisioned automatically. They wouldn’t need to know exactly which resources they needed to provision or how to connect them together. We identified three initial common microservice patterns for containerized applications that we wanted to build architecture as code tooling around: API Service, Queue Processor, and Scheduled Job (all seen in the voting application diagram above).

We chose to distribute our initial set of ECS architectural patterns in the [AWS Cloud Development Kit (AWS CDK)](https://aws.amazon.com/cdk/). AWS CDK is an open source software development framework to model your cloud application resources using familiar programming languages like TypeScript, Python, Java, and C#, and then provision them with CloudFormation. Within the CDK, we built the [“ECS Patterns” module](https://docs.aws.amazon.com/cdk/api/v2/), combining lower-level individual resources into higher-level abstracted resource types that represent an entire containerized microservice. We started by releasing three patterns for ECS: Load Balanced Service, Queue Processing Service, and Scheduled Tasks.

With these patterns, I can model and provision the voting application architecture diagram above, without digging into the nitty-gritty of the individual infrastructure resource configuration. I can model my complete Vote API service with only 36 lines of TypeScript code, instead of 775 lines of raw CloudFormation template! The “[ApplicationLoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/v2/)” class seen below is one of the ECS patterns in the CDK. With this code, I get an ECS service running on Fargate, an Application Load Balancer, an ECS cluster, an ECR repository containing my image, a VPC, a DNS entry, and a TLS certificate, all wired together for me.

```
import { ContainerImage } from '@aws-cdk/aws-ecs';
import { ApplicationLoadBalancedFargateService } from '@aws-cdk/aws-ecs-patterns';
import { ApplicationProtocol } from '@aws-cdk/aws-elasticloadbalancingv2';
import { HostedZone } from '@aws-cdk/aws-route53';
import cdk = require('@aws-cdk/core');
import path = require('path');

// Stack with a Fargate service + load balancer serving https://api.my-vote-app.com
class VoteApiStack extends cdk.Stack {
  constructor(parent: cdk.App, name: string, props: cdk.StackProps) {
    super(parent, name, props);

    // Domain name info
    const domainName = 'api.my-vote-app.com';
    const domainZone = HostedZone.fromLookup(this, 'Zone', {
      domainName: 'my-vote-app.com'
    });

    new ApplicationLoadBalancedFargateService(this, 'Service', {
      protocol: ApplicationProtocol.HTTPS,
      domainName,
      domainZone,
      taskImageOptions: {
        // build and push image using Dockerfile located in vote-api directory
        image: ContainerImage.fromAsset(path.resolve(__dirname, '../vote-api/')),
      },
      desiredCount: 3
    });
  }
}

const app = new cdk.App();
new VoteApiStack(app, 'VoteApiService', {
  env: { account: process.env['CDK_DEFAULT_ACCOUNT'], region: process.env['CDK_DEFAULT_REGION'] }
});
app.synth();
```

## Conclusion

This post walked through the benefits of infrastructure as code for containerized applications, Amazon’s own best practices for deploying containers with infrastructure as code, the concept of “architecture as code,” and an example of architecture as code with Amazon ECS and the AWS CDK. To try out practicing architecture as code with containerized applications on AWS, check out [the AWS CDK ECS patterns](https://docs.aws.amazon.com/cdk/api/v2/).

We are continuing to explore how we can enable our AWS Containers customers to use infrastructure as code and architecture as code through the AWS CDK and through other tools. We recently put up a [public proposal](https://github.com/aws/containers-roadmap/issues/513) with our plans for a new ECS CLI tool designed around the architecture as code model (feedback is welcome!). We would love to hear your feedback on the CDK ECS patterns [on GitHub](https://github.com/aws/aws-cdk/issues/new/choose). Tell us what patterns are missing, for example what other architecture and microservice patterns you see in your own containerized applications that could be captured in re-usable infrastructure as code components. And, let us know what other infrastructure as code tooling and integrations you would like to see from the Containers team on the [AWS Containers Roadmap](https://github.com/aws/containers-roadmap).

> TAGS: [Amazon ECS](https://aws.amazon.com/blogs/containers/tag/amazon-ecs/), [Amazon EKS](https://aws.amazon.com/blogs/containers/tag/amazon-eks/), [infrastructure as code](https://aws.amazon.com/blogs/containers/tag/infrastructure-as-code/)