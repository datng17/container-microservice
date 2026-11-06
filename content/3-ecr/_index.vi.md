---
title: "Tạo ECR Image Repository bằng AWS CDK và push Docker image"
date: "2026-09-30"
weight: 3
chapter: true
pre: "<b>3.</b>"
---

# Tạo ECR và push Docker image

Trong phần này, chúng ta sẽ tìm hiểu về cách xây dựng hạ tầng tạo kho lưu trữ **Amazon Elastic Container Registry (ECR)** trên AWS bằng **AWS CDK (TypeScript)**. Đồng thời, chúng ta sẽ thực hiện cấu hình xác thực, gắn thẻ (tag) và đẩy (push) Docker image của ứng dụng Spring Boot lên ECR repository.

![ECR Repo and Push Docker image](/images/UPDATE/03-ecr/1.png)
