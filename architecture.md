# DevOps Challenge: Healthcare Microservices Deployment on AWS Fargate

## Table of Contents

1.  [Overview](#1-overview)
2.  [Architecture](#2-architecture)
3.  [Services](#3-services)
4.  [Technology Stack](#4-technology-stack)
5.  [Project Structure](#5-project-structure)
6.  [Setup and Prerequisites](#6-setup-and-prerequisites)
    * [AWS Configuration](#aws-configuration)
    * [Local Machine Setup](#local-machine-setup)
    * [GitHub Repository Setup](#github-repository-setup)
7.  [Deployment Workflows (GitHub Actions)](#7-deployment-workflows-github-actions)
    * [Infrastructure as Code (Terraform) Workflow](#infrastructure-as-code-terraform-workflow)
    * [Application CI/CD Workflow](#application-cicd-workflow)
8.  [Monitoring and Logging](#8-monitoring-and-logging)
9.  [Cleanup](#9-cleanup)
10. [Evaluation Criteria](#10-evaluation-criteria)
11. [Contact](#11-contact)

## 1. Overview

This project demonstrates a comprehensive DevOps pipeline for deploying a simple healthcare application consisting of two Node.js microservices (Patient Service and Appointment Service) on AWS Fargate. The solution leverages Infrastructure as Code (IaC) with Terraform, containerization with Docker, and automated CI/CD pipelines using GitHub Actions.

**Key Objectives:**

* Containerize Node.js microservices.
* Provision AWS infrastructure using Terraform with remote state management.
* Implement robust CI/CD pipelines for both infrastructure and application code.
* Deploy microservices to AWS Fargate, a serverless container orchestration service.
* Configure basic monitoring and logging using AWS CloudWatch.

## 2. Architecture

The architecture is designed for high availability, scalability, and automation.

## 3. Services
The application consists of two Node.js microservices:
Patient Service: Manages patient-related information.
Appointment Service: Manages appointment scheduling and details.

## 4. Technology Stack
Cloud Provider: AWS
Containerization: Docker
Container Orchestration: AWS Fargate (via ECS)
Infrastructure as Code (IaC): Terraform
CI/CD: GitHub Actions
Monitoring & Logging: AWS CloudWatch (Logs, Container Insights, Dashboards)
Load Balancing: AWS Application Load Balancer (ALB)
Source Code Management: GitHub
Application Language: Node.js

## 5. Setup and Prerequisites
Before you begin, ensure you have the following configured:
AWS Configuration
AWS Account: An active AWS account is required.
AWS CLI: Install and configure the AWS CLI with appropriate credentials. Ensure your default region is set.
AWS Account: An active AWS account is required.

AWS CLI: Install and configure the AWS CLI with appropriate credentials. Ensure your default region is set.

aws configure
S3 Bucket for Terraform State: Manually create an S3 bucket for Terraform remote state storage. Choose a unique name (e.g., your-name-terraform-state-bucket).

Note: These resources are created manually to ensure they exist before Terraform attempts to use them as a backend. The terraform/modules/state_backend module can later manage these if desired, but for initial setup, manual creation is safer.

**Local Machine Setup**
Git: Install Git.
Terraform: Install Terraform (v1.0+ recommended).
Docker: Install Docker Desktop (or Docker Engine).
Node.js & npm: (Optional, for local development/testing of microservices).

GitHub Repository Setup
Fork/Clone Repository: Fork this repository and clone it to your local machine.


git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
cd your-repo-name
GitHub Secrets: In your GitHub repository settings, navigate to Settings > Secrets and variables > Actions > New repository secret. Add the following secrets:
AWS_REGION: Your preferred AWS region (e.g., ap-south-1).
AWS_ACCOUNT_ID: Your AWS account ID.
TF_STATE_BUCKET: The name of the S3 bucket you created for Terraform state (e.g., your-name-terraform-state-bucket).
OIDC Configuration (Recommended): To enhance security and avoid storing long-lived AWS credentials in GitHub Secrets, it's highly recommended to configure OpenID Connect (OIDC) between GitHub Actions and AWS.
Follow the official AWS documentation to set up an IAM OIDC provider and an IAM role that GitHub Actions can assume. This role should have permissions equivalent to AdministratorAccess for initial setup, or fine-grained permissions for specific resources later.
Update terraform/modules/iam/main.tf to provision the necessary OIDC provider and role.
Once configured, your GitHub Actions workflows (.github/workflows/*.yml) will use aws-actions/configure-aws-credentials@v4 with role-to-assume to authenticate.
Example (simplified for your terraform/environments/dev/main.tf to reference the backend config):
Terraform

terraform {
  backend "s3" {
    bucket         = "your-name-terraform-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1" # Or your chosen region
    encrypt        = true
    dynamodb_table = "terraform-state-locking"
  }
}


Make sure to update the bucket and region in your main.tf files within terraform/environments/dev, staging, and prod to match your S3 bucket and desired region.

## 6. Deployment Workflows (GitHub Actions)
This project utilizes GitHub Actions for automated CI/CD.
Infrastructure as Code (Terraform) Workflow (.github/workflows/terraform.yml)
This workflow manages the AWS infrastructure.
Triggers:
pull_request: On creation or update of pull requests targeting the main branch.
push: On pushes to the main branch.

Jobs:

terraform_fmt_validate (on PRs):
Checks Terraform code formatting (terraform fmt --check).
Validates Terraform configuration (terraform validate).
terraform_plan (on PRs):
Generates a terraform plan and uploads it as an artifact. This allows reviewers to see the infrastructure changes before merging.
terraform_apply (on merges to main):
Runs terraform apply -auto-approve to provision or update the AWS infrastructure.
This step is conditional and only runs on successful merges to the main branch, ensuring a controlled deployment
The workflow uses terraform workspace select to manage dev, staging, and prod environments, driven by branch names or manual input (not fully implemented in this template, but extendable). For simplicity, this template assumes direct pushes/merges to main deploy to a dev workspace, and you would extend it for staging/prod branches.
Application CI/CD Workflow (.github/workflows/app-cicd.yml)
This workflow builds, pushes, and deploys the microservices.

Triggers:
push: On pushes to the main branch (for a unified dev environment deployment).

Jobs:

build_and_push_docker_images:
Authenticates with ECR using OIDC.
Builds Docker images for patient-service and appointment-service.
Tags and pushes the images to their respective ECR repositories.
deploy_to_ecs_fargate:
Authenticates with AWS.
Updates the ECS task definitions with the newly pushed image tags.
Updates the ECS services to use the latest task definition, triggering a deployment.

## 7. Monitoring and Logging
AWS CloudWatch Logs:
All container logs from Fargate tasks are automatically sent to dedicated CloudWatch Log Groups (e.g., /ecs/patient-service, /ecs/appointment-service).

AWS CloudWatch Container Insights:
Enabled on the ECS cluster to provide detailed performance metrics and diagnostic information for containers, tasks, and services (CPU, memory, network, storage).

Custom CloudWatch Dashboards (Bonus):
(To be implemented) Create custom dashboards to visualize key metrics like ALB request count, latency, HTTP error rates, ECS task CPU/memory utilization, and application-specific metrics from logs.

To view logs and metrics:
Navigate to the CloudWatch console in your AWS account.
For logs, go to Log groups and search for /ecs/.
For metrics, go to Metrics -> ECS or Container Insights.

## 8. Cleanup
To avoid incurring unexpected AWS costs, ensure you clean up all resources after you are done.
Destroy Terraform Infrastructure:

