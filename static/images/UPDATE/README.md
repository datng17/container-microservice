# UPDATE image checklist

Thư mục này dành cho hình ảnh của bản lab ECS CDK mới. Ảnh thật sẽ được bổ sung sau khi nội dung và thao tác trên AWS được chốt.

Giữ tên file ổn định và không chụp access key, secret, token hoặc dữ liệu cá nhân. Nếu ảnh có account ID, hãy che hoặc làm mờ trước khi xuất bản.

Các thư mục cần tạo ảnh:

- 01-architecture: sơ đồ ECR, VPC, ECS, ALB, NLB, VPC Link, API Gateway và DynamoDB.
- 02-environment: phiên bản JDK, Node.js, npm, AWS CLI và Docker.
- 03-springboot: cấu trúc source, Maven build, health endpoint và API local.
- 04-docker: Dockerfile, build image, image local và container chạy.
- 05-cdk: npm ci, cdk list, cdk bootstrap và cdk synth.
- 06-ecr: repository, Docker login, tag và image đã push.
- 07-network: VPC, subnet, route table, NAT Gateway và security group.
- 08-ecs: cluster, task definition, service, task running và ALB target health.
- 09-api: NLB listener, VPC Link, HTTP API route và API endpoint.
- 10-dynamodb: bảng products và các request CRUD.
- 11-observability: CloudWatch log stream, X-Ray sidecar và Trace Map.
- 12-update: image tag mới và ECS task definition revision.
- 13-cleanup: cdk destroy và xác nhận tài nguyên đã xóa.

