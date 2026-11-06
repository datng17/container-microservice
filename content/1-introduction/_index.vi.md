---
title: "Giới thiệu kiến trúc và dịch vụ"
date: "2026-09-30"
weight: 1
chapter: true
pre: "<b>1.</b>"
---

# Giới thiệu kiến trúc và dịch vụ

## Mục tiêu của bài lab

Mục tiêu của bài lab là giúp người học hiểu được toàn bộ đường đi từ mã nguồn ứng dụng đến một dịch vụ REST đang chạy trên AWS. Ta sẽ bắt đầu với **Spring Boot** và **Docker**, sau đó dùng **ECR** làm kho lưu trữ image, **ECS Fargate** làm nơi chạy container, **VPC** làm mạng riêng, **DynamoDB** làm cơ sở dữ liệu và **API Gateway** làm cửa vào public.

## AWS CDK là gì?

**AWS Cloud Development Kit**, hay **AWS CDK**, là cách mô tả hạ tầng AWS bằng ngôn ngữ lập trình. Trong bài này, **TypeScript** mô tả các **stack** và **resource**. Lệnh `cdk synth` chuyển **TypeScript** thành template **CloudFormation**, còn `cdk deploy` yêu cầu **CloudFormation** tạo hoặc cập nhật các resource đó.

Có thể hình dung CDK như bản vẽ có thể chạy lại. Thay vì tạo thủ công từng VPC, subnet, ECS service hay API Gateway trên AWS Management Console, ta lưu bản vẽ trong mã nguồn. Khi mã nguồn thay đổi, CDK tạo ra một phiên bản template mới để CloudFormation so sánh và triển khai.

## Cấu trúc thư mục của repository

Repository được chia thành hai phần rõ ràng: 
- Thư mục **fcaj-spring** chứa mã nguồn ứng dụng và **Dockerfile**. 

- Thư mục **cdk** chứa toàn bộ mã TypeScript dùng để mô tả hạ tầng AWS.

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

File **bin/cdk.ts** là điểm bắt đầu của CDK. File này tạo các stack và truyền resource giữa chúng. Ví dụ, **VpcStack** cung cấp **VPC** cho **EcsStack**, **DynamoDbStack** cung cấp bảng **products** cho **task role**, **EcsStack** cung cấp **ALB listener** cho **NlbStack**, còn **NlbStack** cung cấp **NLB listener** cho **ApiGatewayStack**.

Khi đọc code, nên đi theo thứ tự **bin/cdk.ts**, **cdk-stack.ts**, **vpc-stack.ts**, **dynamodb-stack.ts**, **ecs-stack.ts**, **nlb-stack.ts** và **api-gate-way-stack.ts**. Sau đó chuyển sang **fcaj-spring** để xem **controller** xử lý request như thế nào và **DynamoDbConfig** tạo **AWS client** ra sao.

## Các tài nguyên và sự liên kết của chúng trong workshop

- **AWS ECR** là kho chứa Docker image.

- **AWS ECS** là dịch vụ điều phối container của AWS, còn **AWS Fargate** là dịch vụ tính toán không máy chủ (compute engine) giúp chạy container mà không cần tự quản lý EC2 -> giảm chi phí vận hành của các ứng dụng dựa trên container. 

- **AWS VPC** cung cấp không gian mạng riêng với public subnet và private subnet.

- **AWS ALB** hiểu giao thức **HTTP** và thực hiện health check tới endpoint `/actuator/health` của **Spring Boot**. AWS **NLB** cung cấp một listener ổn định cho **private integration**. **AWS VPC Link** tạo đường kết nối riêng từ **API Gateway** vào **VPC**. **AWS API Gateway** nhận request từ bên ngoài rồi chuyển request qua **VPC Link** tới **NLB**, **ALB** và **ECS task**.

- **AWS IAM** là service quản lý quyền truy cập tới tài nguyên AWS. Bao gồm: 
    * **Task execution role**: pull image từ **ECR** và gửi log tới **CloudWatch**.
    * **Task role**: cho ứng dụng **Spring Boot** gọi **DynamoDB** và **X-Ray**.

- **AWS DynamoDB** là một dịch vụ quản lý cơ sở dữ liệu NoSQL mạnh mẽ và phi quan hệ. Bài lab này giới thiệu việc sử dụng **DynamoDB enhanced client** từ **AWS SDK V2 cho Java**, đây là một thư viện cấp cao cho phép ánh xạ các lớp phía máy khách với các bảng **DynamoDB**.

- **AWS CloudWatch Logs** tập trung nhật ký (log) của container và ứng dụng. Các ứng dụng được tạo trong khóa học này sẽ ghi log vào **CloudWatch Logs** theo định dạng **JSON**, sử dụng thư viện **Log4j2**. Điều này cho phép chèn các tham số vào log để sử dụng trong các truy vấn tại bảng điều khiển **AWS CloudWatch Logs Insights**.

- **AWS X-Ray** daemon sidecar thu thập, gom nhóm và gửi dữ liệu trace (dấu vết yêu cầu) từ container ứng dụng đến dịch vụ **AWS X-Ray** qua mạng nội bộ (**UDP**).

## Luồng xử lý một request

Khi người dùng gọi **GET /api/products**, request đi vào **API Gateway**. API Gateway chuyển request qua **VPC Link** tới listener của **NLB**. NLB chuyển tiếp tới **internal ALB**. ALB chọn một ECS task đang khỏe dựa trên health check. **Spring Boot** xử lý request, gọi **DynamoDB** và trả response. Log được gửi tới **CloudWatch**, còn trace được chuyển tới **X-Ray** daemon chạy cạnh container ứng dụng.

**ALB** là internal (nằm trong private subnet) nên người dùng Internet không gọi trực tiếp **DNS** của **ALB**. Đây là lý do bài lab có **API Gateway** và **VPC Link** ở phía trước.
