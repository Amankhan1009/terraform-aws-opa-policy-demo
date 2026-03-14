# Terraform AWS OPA Policy Demo

## Overview

This project demonstrates how to enforce **Infrastructure as Code (IaC) security policies** using **Terraform, Open Policy Agent (OPA), and Conftest**.
It integrates **GitHub Actions CI/CD** with **AWS IAM OIDC authentication** to securely deploy infrastructure.

The pipeline validates Terraform code, runs policy checks using OPA, and only applies the infrastructure if the policies pass.

---

## Architecture

GitHub → GitHub Actions → OIDC Authentication → AWS IAM Role → Terraform → EC2 Infrastructure

---

## Technologies Used

* Terraform
* AWS (EC2, IAM)
* Open Policy Agent (OPA)
* Conftest
* GitHub Actions
* AWS OIDC Federation

---

## Project Structure

```
terraform-aws-opa-policy-demo
│
├── terraform
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── policy
│   └── policy.rego
│
└── .github
    └── workflows
        └── terraform.yml
```

---

## Policy Example

The OPA policy ensures only **t2.micro instance types** are allowed.

```
package main

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_instance"
  resource.change.after.instance_type != "t2.micro"

  msg := "Only t2.micro instance type is allowed"
}
```

If another instance type is used, the pipeline will fail.

---

## CI/CD Workflow

The GitHub Actions pipeline performs the following steps:

1. Checkout repository
2. Authenticate to AWS using OIDC
3. Initialize Terraform
4. Validate Terraform configuration
5. Generate Terraform execution plan
6. Convert Terraform plan to JSON
7. Run OPA policy checks with Conftest
8. Apply infrastructure if policy checks pass

---

## GitHub Actions Workflow

Workflow file location:

```
.github/workflows/terraform.yml
```

Trigger:

```
Push to main branch
```

---

## AWS Setup

### 1. Create OIDC Provider

Add GitHub as an identity provider in AWS IAM.

Provider URL:

```
https://token.actions.githubusercontent.com
```

Audience:

```
sts.amazonaws.com
```

---

### 2. Create IAM Role for GitHub Actions

Trust policy example:

```
repo:USERNAME/terraform-aws-opa-policy-demo:*
```

Attach permissions such as:

* AmazonEC2FullAccess
* IAM role permissions required for Terraform

---

## Running Locally

Initialize Terraform:

```
terraform init
```

Validate configuration:

```
terraform validate
```

Create plan:

```
terraform plan -out=tfplan
```

Convert plan to JSON:

```
terraform show -json tfplan > tfplan.json
```

Run policy test:

```
conftest test tfplan.json --policy policy
```

---

## Security Benefits

* Prevents unsafe infrastructure changes
* Enforces policy compliance before deployment
* Removes need for long-lived AWS credentials
* Uses secure OIDC authentication

---

## Example Use Case

If a developer tries to deploy:

```
instance_type = "t3.medium"
```

The pipeline will fail with:

```
FAIL - Only t3.micro instance type is allowed
```

---

## Future Improvements

* Add multiple policy rules
* Integrate Terraform security scanners
* Add pull request approval workflow
* Deploy infrastructure using modules

---

## Author

Md Aman Alam

---

## License

This project is for educational and demonstration purposes.
