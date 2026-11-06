---
title: "Deploy Stack to AWS ECR"
date: "2026-09-30"
weight: 3
chapter: false
pre: "<b>3.3. </b>"
---

In this step, we will deploy the **ProductsServiceEcrStack** to our AWS account using the AWS CDK CLI, monitor the CloudFormation resource creation process, and verify the ECR repository in the AWS Management Console.

---

### 1. Check Environment Bootstrap

Before any stack can be deployed, the AWS environment (Account & Region) must be bootstrapped with management resources (S3 bucket for assets, CDK IAM roles).

Check account information:

~~~powershell
aws sts get-caller-identity --output text
~~~ 

Run the bootstrap command:

~~~powershell
npx cdk bootstrap
~~~

![Check bootstrap](/images/UPDATE/03-ecr/9.png)

#### Access CloudFormation to check the CDKToolkit stack.

Access the AWS Console -> CloudFormation -> Stacks -> Select stack `CDKToolkit`.

![Access CloudFormation to check created stack](/images/UPDATE/03-ecr/10.png)

![Check CDKToolkit stack](/images/UPDATE/03-ecr/11.png)

![CDKToolkit stack check result](/images/UPDATE/03-ecr/12.png)

---

### 2. Deploy ECR Stack (`cdk deploy`)

1. Open Terminal in VS Code at the **IaC-ECS-CDK/cdk** directory.

2. Run the command to deploy only the ECR Stack:

~~~powershell
npx cdk deploy ProductsServiceEcrStack --require-approval never
~~~

{{% notice info %}}
**Parameter meaning:**  
The **--require-approval never** parameter directs CDK to automatically approve security-related changes (such as IAM Roles or Resource Policies) without stopping to ask the user to type **y/n**, allowing the deployment process to run smoothly.
{{% /notice %}}

3. The deployment process will create a CloudFormation ChangeSet and provision the **AWS::ECR::Repository** resource. When completed, the terminal will display a success message along with the **Outputs** information:

![Run cdk deploy](/images/UPDATE/03-ecr/13.png)

{{% notice tip %}}
Save the **RepositoryUri** value displayed in the Outputs section. This is the unique registry identifier on AWS ECR, which will be used for tagging and pushing the Docker image in the next step.
{{% /notice %}}

---

#### Access CloudFormation to check the ProductsServiceEcrStack.

![Check ProductsServiceEcrStack](/images/UPDATE/03-ecr/14.png)

![ProductsServiceEcrStack check result](/images/UPDATE/03-ecr/15.png)

### 3. Verify ECR in the AWS Management Console

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).

2. In the top search bar, type **ECR** and select the **Elastic Container Registry** service. Ensure you have selected the correct region **Singapore (ap-southeast-1)** in the upper right corner.

![Search ECR service on Console](/images/UPDATE/03-ecr/16.png)

3. In the left navigation menu, select **Repositories** (under **Private registry**). You will see the newly created repository named **`fcaj-products-service`** appear in the list.

![ECR Repositories list](/images/UPDATE/03-ecr/17.png)

4. Click on the repository name **`fcaj-products-service`** to view details.

![fcaj-products-service repository details](/images/UPDATE/03-ecr/18.png)

Currently, the repository is newly created, so there are no images yet. We will proceed to authenticate Docker and push the image to this repository in the next step.
