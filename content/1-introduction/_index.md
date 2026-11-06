---
title: "Introduction to Architecture and Services"
date: "2026-09-30"
weight: 1
chapter: true
pre: "<b>1.</b>"
---

# Introduction to Architecture and Services

## Lab Objectives

The objective of this lab is to help learners understand the entire journey from application source code to a running REST service on AWS. We will start with **Spring Boot** and **Docker**, then use **ECR** as the image repository, **ECS Fargate** as the container runtime, **VPC** as the private network, **DynamoDB** as the database, and **API Gateway** as the public entry point.

## What is AWS CDK?

AWS Cloud Development Kit, or **AWS CDK**, is a framework for defining **AWS** infrastructure using programming languages. In this lab, **TypeScript** describes the **stack** and resources. The `cdk synth` command synthesizes **TypeScript** into **CloudFormation** templates, while `cdk deploy` requests **CloudFormation** to create or update those resources.

You can think of CDK as an executable blueprint. Instead of manually creating each VPC, subnet, ECS service, or API Gateway in the AWS Management Console, we store the blueprint in source code. When the source code changes, CDK generates a new template version for CloudFormation to compare and deploy.

## Repository Directory Structure

The repository is divided into two distinct parts:
- The **fcaj-spring** directory contains the application source code and **Dockerfile**.
- The **cdk** directory contains all **TypeScript** code used to describe **AWS** infrastructure.

~~~text
IaC-ECS-CDK/
├─ fcaj-spring/                         # Spring Boot and Dockerfile
│  ├─ pom.xml                            # Maven dependencies and configuration
│  ├─ Dockerfile                         # Docker image recipe
│  └─ src/main/java/com/example/
│     └─ fcaj_spring/products/           # Product model, controller and AWS config
└─ cdk/                                  # AWS infrastructure in TypeScript
   ├─ bin/cdk.ts                         # CDK entrypoint: initializes and connects stacks
   └─ lib/
      ├─ cdk-stack.ts                    # Creates ECR repository
      ├─ vpc-stack.ts                    # Creates VPC and subnets
      ├─ dynamodb-stack.ts               # Creates products table
      ├─ ecs-stack.ts                    # ECS, Fargate, task definition and internal ALB
      ├─ nlb-stack.ts                    # Internal NLB forwarding to ALB
      └─ api-gate-way-stack.ts           # HTTP API and VPC Link
~~~

The **bin/cdk.ts** file is the entry point of the **CDK** application. This file instantiates **stacks** and passes **resources** between them. For example, **VpcStack** provides the **VPC** to **EcsStack**, **DynamoDbStack** provides the products table to the **task role**, **EcsStack** provides the **ALB listener** to **NlbStack**, and **NlbStack** provides the **NLB listener** to **ApiGatewayStack**.

When reading the code, you should follow the sequence: **bin/cdk.ts**, **cdk-stack.ts**, **vpc-stack.ts**, **dynamodb-stack.ts**, **ecs-stack.ts**, **nlb-stack.ts**, and **api-gate-way-stack.ts**. Then switch to **fcaj-spring** to see how the **controller** handles requests and how **DynamoDbConfig** creates the **AWS client**.

## Resources and Their Relationships in the Workshop

- **AWS ECR** is the Docker image registry.

- **AWS ECS** is the container orchestration service from AWS, while **AWS Fargate** is a serverless compute engine that runs containers without having to manage EC2 instances -> reducing operational overhead for containerized applications.

- **AWS VPC** provides private network space with public and private subnets.

- **AWS ALB** understands the **HTTP protocol** and performs health checks against the `/actuator/health` endpoint of Spring Boot. AWS NLB provides a stable listener for private integration. AWS VPC Link creates a private connection from API Gateway into the VPC. AWS API Gateway receives external requests and forwards them via VPC Link to NLB, ALB, and the ECS tasks.

- **AWS IAM** is a service that provides access control to AWS resources. In this workshop, it includes:
    * Task execution role: pull image from ECR and send logs to CloudWatch.
    * Task role: for the Spring Boot application to call DynamoDB and X-Ray.

- **AWS DynamoDB** is a powerful non-relational **NoSQL** database management service. This lab introduces the use of **DynamoDB Enhanced Client** from **AWS SDK v2 for Java**, a high-level library that maps client-side classes to **DynamoDB** tables.

- **AWS CloudWatch Logs** centralizes container and application logs. The applications created in this course write logs to CloudWatch Logs in JSON format using the **Log4j2** library. This allows inserting parameters into logs to be used in queries within the **AWS CloudWatch Logs Insights** console.

- The **AWS X-Ray** daemon sidecar collects, batches, and sends trace data (request traces) from the application container to the **AWS X-Ray** service over the internal network (UDP).

## Request Processing Flow

When a user calls **GET /api/products**, the request reaches **API Gateway**. API Gateway forwards the request through the **VPC Link** to the **NLB** listener. **NLB** forwards it to the internal **ALB**. **ALB** selects a healthy **ECS** task based on health checks. **Spring Boot** processes the request, queries **DynamoDB**, and returns the response. Logs are sent to **CloudWatch**, while traces are sent to the **X-Ray** daemon running alongside the application container.

Because the **ALB** is internal (inside private subnet), internet users cannot call the **ALB** **DNS** directly. This is why the lab places **API Gateway** and **VPC Link** in front.
