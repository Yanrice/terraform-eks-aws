# AWS EKS Cluster Terraform Deployment with Jenkins Pipeline

This repository contains Terraform configuration to deploy an EKS cluster and node group in AWS, along with a Jenkins pipeline to automate deployment and optional destruction.

## Prerequisites

* Terraform installed (`v1.5.0` or later recommended)
* AWS CLI configured with appropriate credentials
* Git installed for pushing to GitHub
* Jenkins instance (e.g., at https://jenkins-yannickkalukuta.com) with Git and Terraform installed on the agents
* AWS account with sufficient permissions for EKS (e.g., IAM roles for EKS)
* S3 bucket and DynamoDB table for Terraform state (optional, recommended)

## Terraform State Management

The Terraform state file is stored in an S3 bucket for reliability and collaboration. The configuration uses:

* S3 Bucket: my-terraform-state-bucket (replace with your bucket name)
* State Path: eks/terraform.tfstate
* DynamoDB Table: terraform-locks for state locking
* Setup:
- Create an S3 bucket with versioning enabled.
- Create a DynamoDB table named terraform-locks with primary key LockID (String).
- Attach IAM permissions to your AWS credentials for S3 and DynamoDB access.
- Run `terraform init` to initialize or migrate the state to S3.

Without the S3 backend, the state file is stored locally in the workspace directory and excluded from Git via .gitignore.

## Setup

- Clone this repository:

`git clone <your-repo-url>`
`cd <repo-name>`


- Initialize Terraform:

`terraform init`


- Update variables in `variables.tf` or create a `terraform.tfvars` file:

`aws_region = "us-east-1"`
`project_name = "my-eks-project"`
`cluster_name = "my-eks-cluster"`
`cluster_version = "1.30"`
`node_min_size = 1`
`node_max_size = 3`
`node_desired_size = 2`
`node_instance_type = "t3.medium"`


- Review the plan:

`terraform plan`

- Apply the configuration:

`terraform apply`

## Outputs
After deployment, Terraform will output:

- `cluster_endpoint`: The endpoint for the EKS control plane
- `cluster_name`: The name of the Kubernetes cluster

## Jenkins Pipeline Setup

- Add the Jenkinsfile to your GitHub repo root, commit, and push.

- Configure AWS Credentials in Jenkins:

* IDs: 'aws-access-key-id' and 'aws-secret-access-key'

- Create a New Pipeline Job in Jenkins:

* Name: e.g., "EKS-Deployment-Pipeline"
* Parameterized: Boolean parameter 'DESTROY' (default false)
* Pipeline script from SCM: Git, your repo URL, branch main, script path Jenkinsfile

- Run the Pipeline:

* For deploy: Run with DESTROY unchecked.
* For destroy: Run with DESTROY checked.


## Notes

* This uses Terraform modules for VPC and EKS for best practices.
* Ensure your AWS region supports the chosen Kubernetes version.
* After deployment, update your kubeconfig: `aws eks update-kubeconfig --name ${cluster_name} --region ${aws_region}`
* The pipeline uses `-auto-approve` for apply/destroy; use caution.
* Add `.gitignore` for `*.tfvars`, `.terraform/`, `terraform.tfstate*`
* Monitor AWS costs, as EKS incurs charges.

## Cleanup
Use the destroy option in the Jenkins pipeline or run `terraform destroy` locally.
