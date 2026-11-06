---
title: "Clean up resources"
date: "2026-09-30"
weight: 10
chapter: true
pre: "<b>10. </b>"
---

# Clean up resources

The lab creates a NAT Gateway, VPC Link, load balancers, ECS task, ECR repository, DynamoDB table, CloudWatch log group, and IAM roles. Some of these resources will continue to incur costs if left running.

Delete all stacks using the command:

```powershell
cd cdk
npx cdk destroy --all --force
```

Before deleting, check whether DynamoDB contains any data that needs to be kept. After the command completes, open CloudFormation, ECS, ECR, DynamoDB, VPC, and API Gateway to confirm that resources have been deleted. The ECR repository and DynamoDB table are configured with DESTROY removal policies for lab purposes.

{{% notice tip %}}

Tip for handling stuck ECR DELETE_FAILED errors:
If the ECR Repository reports an error that it cannot be deleted because it still contains images, run the force delete command from the AWS CLI:

{{% /notice %}}

1. Access Cloudformation Console -> select the stuck stack.

![Cloudformation Console](/images/UPDATE/10-cleanUp/1.png)

2. Or use AWS CLI to remove the stuck stack:

```powershell
aws ecr delete-repository --repository-name fcaj-products-service --region ap-southeast-1 --force
```


