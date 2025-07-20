graph TD
    subgraph GitHub
        GH_REPO[GitHub Repository] --> |Push Code| GHA_CI[GitHub Actions CI Workflow]
        GH_REPO --> |PR, Merge| GHA_TF[GitHub Actions IaC Workflow]
    end

    subgraph AWS Cloud
        subgraph VPC (10.0.0.0/16)
            direction LR
            IGW[Internet Gateway] --> PublicSubnetAZ1[Public Subnet AZ1 (10.0.0.0/24)]
            IGW --> PublicSubnetAZ2[Public Subnet AZ2 (10.0.1.0/24)]

            PublicSubnetAZ1 --> ALB[Application Load Balancer]
            PublicSubnetAZ2 --> ALB

            ALB --> PrivateSubnetAZ1[Private Subnet AZ1 (10.0.10.0/24)]
            ALB --> PrivateSubnetAZ2[Private Subnet AZ2 (10.0.11.0/24)]

            PrivateSubnetAZ1 --> FargateTasks[ECS Fargate Tasks]
            PrivateSubnetAZ2 --> FargateTasks

            FargateTasks --> CloudWatchLogs[CloudWatch Logs]
            FargateTasks --> CloudWatchMetrics[CloudWatch Metrics (Container Insights)]

            PrivateSubnetAZ1 --> NATGW1[NAT Gateway AZ1]
            PrivateSubnetAZ2 --> NATGW2[NAT Gateway AZ2]

            NATGW1 --> IGW
            NATGW2 --> IGW
        end

        subgraph S3 & DynamoDB
            S3[S3 Backend (Terraform State)]
            DDB[DynamoDB (Terraform State Locking)]
        end

        subgraph ECR
            ECR_Patient[ECR Repo: Patient Service]
            ECR_Appointment[ECR Repo: Appointment Service]
        end

        subgraph IAM
            IAM_ECS_Task_Role[ECS Task Role]
            IAM_ECS_Exec_Role[ECS Task Execution Role]
            IAM_GHA_Role[GitHub Actions OIDC Role]
        end

        subgraph ECS Cluster
            ECS_CLUSTER[ECS Cluster]
            ECS_CLUSTER --> ECS_Service_Patient[ECS Service: Patient Service]
            ECS_CLUSTER --> ECS_Service_Appointment[ECS Service: Appointment Service]
            ECS_Service_Patient --> ECS_TaskDef_Patient[ECS Task Definition: Patient Service]
            ECS_Service_Appointment --> ECS_TaskDef_Appointment[ECS Task Definition: Appointment Service]
        end
    end

    GHA_CI --> ECR_Patient
    GHA_CI --> ECR_Appointment
    GHA_CI --> ECS_Service_Patient
    GHA_CI --> ECS_Service_Appointment

    GHA_TF --> VPC
    GHA_TF --> S3
    GHA_TF --> DDB
    GHA_TF --> ECR_Patient
    GHA_TF --> ECR_Appointment
    GHA_TF --> IAM
    GHA_TF --> ECS_CLUSTER
    GHA_TF --> ALB
    GHA_TF --> CloudWatchLogs
    GHA_TF --> CloudWatchMetrics

    ECS_TaskDef_Patient --> ECR_Patient
    ECS_TaskDef_Appointment --> ECR_Appointment

    ALB --> ECS_Service_Patient
    ALB --> ECS_Service_Appointment

    subgraph Developer
        Dev[Developer] --> |Code Changes| GH_REPO
    end
