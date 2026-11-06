---
title: "Tạo Spring Boot Product Service"
date: "2026-09-30"
weight: 3
chapter: false
pre: "<b>2.3. </b>"
---

Để triển khai các tài nguyên AWS trên Cloud (ECR, ECS, DynamoDB, APIs, v.v.), chúng ta sẽ sử dụng **AWS Cloud Development Kit (CDK)**. CDK cho phép bạn mô tả hạ tầng bằng code và tự động tạo ra các CloudFormation template để triển khai.

{{% notice tip %}}
Bài workshop này sẽ sử dụng Java Spring Boot cho ứng dụng backend (ECR) và TypeScript cho ECS, VPC, DynamoDB, APIs, v.v. Ngoài ra bạn có thể tạo bằng Python hoặc ngôn ngữ lập trình khác.
{{% /notice %}}

### Cấu trúc thư mục của repository

Repository được chia thành hai phần rõ ràng: 

- Thư mục `fcaj-spring` chứa mã nguồn ứng dụng và Dockerfile. 
- Thư mục `cdk` chứa toàn bộ mã TypeScript dùng để mô tả hạ tầng AWS.

~~~text
IaC-ECS-CDK/
├─ fcaj-spring/                         # Spring Boot và Dockerfile
│  ├─ pom.xml                            # Dependency và cấu hình Maven
│  ├─ Dockerfile                         # Công thức tạo Docker image
│  └─ src/main/java/com/example/
│     └─ fcaj_spring/products/           # Product model, controller và AWS config
└─ cdk/                                  # Hạ tầng AWS bằng TypeScript
   ├─ bin/cdk.ts                         # CDK entrypoint: Khởi tạo và kết nối các stack
   └─ lib/
      ├─ cdk-stack.ts                    # Tạo ECR repository
      ├─ vpc-stack.ts                    # Tạo VPC và subnet
      ├─ dynamodb-stack.ts               # Tạo bảng products
      ├─ ecs-stack.ts                    # ECS, Fargate, task definition và internal ALB
      ├─ nlb-stack.ts                    # Internal NLB chuyển tiếp tới ALB
      └─ api-gate-way-stack.ts           # HTTP API và VPC Link
~~~
