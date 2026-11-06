---
title: "Dọn dẹp tài nguyên"
date: "2026-09-30"
weight: 10
chapter: true
pre: "<b>10. </b>"
---

# Dọn dẹp tài nguyên

Lab tạo NAT Gateway, VPC Link, load balancer, ECS task, ECR repository, DynamoDB table, CloudWatch log group và các IAM role. Một số tài nguyên tiếp tục phát sinh chi phí nếu để lại.

Xóa toàn bộ stack bằng lệnh:

```powershell
cd cdk
npx cdk destroy --all --force
```

Trước khi xóa, kiểm tra DynamoDB có dữ liệu cần giữ hay không. Sau khi lệnh hoàn tất, mở CloudFormation, ECS, ECR, DynamoDB, VPC và API Gateway để xác nhận tài nguyên đã được xóa. Repository ECR và bảng DynamoDB được cấu hình DESTROY cho mục đích lab.

{{% notice tip %}}

Mẹo xử lý khi gặp lỗi kẹt ECR DELETE_FAILED:
Nếu ECR Repository báo lỗi không thể xóa vì còn chứa image, hãy chạy lệnh xóa cưỡng chế.
{{% /notice %}}

1. Truy cập vào Cloudformation -> chọn Stack cần xoá.

![Cloudformation Console](/images/UPDATE/10-cleanUp/1.png)

2. Chạy lệnh AWS CLI để thực hiện xoá cưỡng chế:

```powershell
aws ecr delete-repository --repository-name fcaj-products-service --region ap-southeast-1 --force
```



