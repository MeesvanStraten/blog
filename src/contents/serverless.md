---
author: Mees van Straten
datetime: 2023-10-12 13:00
title: Exploring AWS Serverless Compute Services
slug: exploring-aws-serverless-compute-services
featured: true
draft: false
tags:
  - AWS
  - serverless
  - fargate
  - lambda
ogImage: ""
description:
  Exploring AWS Serverless Compute Services.
---
This post originally appeared on [cloudlegends.nl](https://cloudlegends.nl/en/insights/aws-serverless-compute/)
<link rel="canonical" href="https://cloudlegends.nl/en/insights/aws-serverless-compute/"/>

# Introduction
In this article, we will explore the various options related to serverless computation and discuss how they can benefit your application and business.
# What is serverless?
Serverless is certainly a buzzword, but it can also be an excellent way to focus more on developing new functionality and spend less time managing (cloud) infrastructure.
Serverless is the concept of abstracting away the underlying infrastructure where your code runs. Instead of paying for a (virtual) server for 100% of its capacity, you only pay for what you use. This is primarily based on factors like execution time, CPU, memory or storage consumed.
The advantage here is that your team does not need to patch the operating system on which your application runs. Additionally,  serverless applications can scale up and down quickly when needed. This can decrease cost when your application has a varying load.
# Serverless on AWS
AWS offers a wide variety of serverless options, from computation and storage to integration services. In this blogpost, we will focus on the two computational services: Fargate and Lambda.
## Fargate
Fargate is AWS’s proprietary serverless compute engine. Fargate allows you to specify how much memory and CPU your applications need without managing the underlying infrastructure. This has the following advantages:
- No overhead of managing clusters or servers; \*
- Better scalability by only defining the needs of your application;

  \* You still need to update the EKS Kubernetes version

Fargate can be used with ECS (Elastic Container Service) and EKS (Elastic Kubernetes Service). More information can be found here.

By setting a scaling policy, you can also configure how your application scales based on load. ECS also has an autoscaling policy that can scale depending on your application’s usage.
Fargate does not require architectural changes to host the application, an existing containerized application can easily be deployed to Fargate.

 Deploying larger applications as a single Lambda does come with some drawbacks like cold startup times and maximum execution time. Thus your system can better be split up in smaller separate functions to fully benefit from all the advantages that Lambda has to offer.

### Limitations
Fargate brings some interesting advantages, but it’s worth noting that there are some use cases when serverless is not the best option. Fargate does not support EBS (Elastic Block Storage); however, EFS (Elastic File system)is supported. The lack of EBS support also indicates that Fargate is best suited for stateless applications.

## Lambda
AWS Lambda is a serverless Function-as-a-Service platform. All you need to do is  deliver your code, and AWS takes care of ensuring it runs. You can deploy your code as a container or with the runtime of your choice such as Node.js, Python, Java, .NET, Go and Ruby.
Lambda operates on an event-driven model, making it a perfect fit for reacting to and processing events, then winding down when no events require processing. While event-driven architectures can be more complex, they also offer great flexibility.

For example, let's say a user of your application uploads a high-resolution image to an S3 bucket, and you want to compress and archive it. The S3 bucket can trigger the Lambda function, which then compresses the image and uploads it back to a different S3 bucket. Afterward, the original high-resolution image can be deleted to save on storage cost.
When a large number of images are uploaded, Lambda can scale out to handle the increased traffic. When no images are uploaded, Lambda incurs no costs, as you pay only for the time in seconds that the function is executed and the number of requests made.
With Lambda, you can specify the amount of memory available to your code. CPU allocation is coupled with memory and can not be set separately.

### Limitations
Lambda is ideal for creating event-driven applications and for integrating systems. For long-running workloads, Lambda is not the best fit, since it has a max execution time of 15 minutes. Stateful workloads, such as WebSockets or long running tasks in general, are better suited for a different service like EC2.
Also your application needs specific AWS Lambda dependencies and code implementation. This means your application is more tightly coupled with AWS. Unlike Fargate where a containerized application can be deployed without code changes.

## Conclusion
AWS has over 200 services, a number that is likely to increase in the future. In this article, we focused on the serverless computation services, specifically Fargate and Lambda. Fargate is an excellent choice for running stateless applications without the hassle of maintaining and configuring infrastructure.

With Fargate, you define your application’s needs for CPU, memory and networking, and AWS handles the infrastructure for you. 
When you want to create an integration between applications or create an event-driven architecture, AWS Lambda can be a compelling service.
With Lambda, you can deploy your code as a container or pick a runtime.

Depending on the requirements, you can specify the amount of memory available to the function.
Lambda is ideal for executing your functions independently without worrying about provisioning infrastructure. When your application requires to continuously run Fargate is a better option since Lambda functions have a maximum execution time of 15 minutes.

Lambda does require code changes and adding dependencies, while Fargate does not require code changes and can just be deployed as a containerized application. Overall Lambda has a higher impact on the architecture while Fargate does not require architectural changes.
