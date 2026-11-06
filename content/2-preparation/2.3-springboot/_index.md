---
title: "Create Spring Boot Product Service"
date: "2026-09-30"
weight: 3
chapter: false
pre: "<b>2.3. </b>"
---

To deploy AWS resources in the Cloud (ECR, ECS, DynamoDB, APIs, etc.), we will use the **AWS Cloud Development Kit (CDK)**. CDK allows you to define infrastructure using code and automatically generates CloudFormation templates for deployment.

{{% notice tip %}}
This workshop uses Java Spring Boot for the backend application (ECR) and TypeScript for ECS, VPC, DynamoDB, APIs, etc. You can also build it using Python or other programming languages.
{{% /notice %}}

### Repository Directory Structure

The repository is divided into two distinct parts:

- The `fcaj-spring` directory contains the application source code and Dockerfile.
- The `cdk` directory contains all TypeScript code used to describe AWS infrastructure.

~~~text
IaC-ECS-CDK/
├─ fcaj-spring/                         # Spring Boot and Dockerfile
│  ├─ pom.xml                            # Maven dependencies and configuration
│  ├─ Dockerfile                         # Docker image recipe
│  └─ src/main/java/com/example/
│     └─ fcaj_spring/products/           # Product model, controller, and AWS config
└─ cdk/                                  # AWS infrastructure in TypeScript
   ├─ bin/cdk.ts                         # Stacks initialization and connection
   └─ lib/
      ├─ cdk-stack.ts                    # Creates ECR repository
      ├─ vpc-stack.ts                    # Creates VPC and subnets
      ├─ dynamodb-stack.ts               # Creates products table
      ├─ ecs-stack.ts                    # ECS, Fargate, task, and internal ALB
      ├─ nlb-stack.ts                    # Internal NLB forwarding to ALB
      └─ api-gate-way-stack.ts           # HTTP API and VPC Link
~~~

