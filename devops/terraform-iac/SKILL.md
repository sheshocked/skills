---
name: terraform-iac
description: 
category: devops
tags: [terraform-iac]
---

## When to Use
Define and manage cloud infrastructure as code with Terraform. Covers providers, modules, state management, workspaces, import, drift detection, and multi-environment deployments.

## Core Concepts
- **Providers**: Plugin for cloud APIs (AWS, GCP, Azure, Cloudflare)
- **Resources**: Infrastructure objects (VMs, VPCs, DNS records)
- **Modules**: Reusable, composable infrastructure components
- **State**: JSON file tracking resource IDs and attributes
- **Data sources**: Read existing resources without managing them
- **Plan/Apply**: Dry-run then execute changes

## Workflow
1. Initialize workspace: `terraform init`
2. Write infrastructure in HCL
3. Plan changes: `terraform plan -out=tfplan`
4. Apply: `terraform apply tfplan`
5. Store state in remote backend (S3, GCS, Terraform Cloud)
6. Use workspaces or directory-based environments

## Key Patterns
```hcl
# main.tf — AWS VPC with subnets
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
    }
  }
}

variable "environment" {
  type    = string
  default = "staging"
  validation {
    condition     = contains(["staging", "production"], var.environment)
    error_message = "Must be staging or production"
  }
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "${var.environment}-vpc"
  cidr = var.environment == "production" ? "10.0.0.0/16" : "10.1.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = var.environment != "production"
  enable_dns_hostnames = true

  tags = {
    Environment = var.environment
  }
}

# Output
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

```hcl
# Import existing resources
# terraform import aws_s3_bucket.logs my-logs-bucket
# terraform import 'module.vpc.aws_subnet.private[0]' subnet-abc123

# Drift detection
# terraform plan -detailed-exitcode  # returns 2 if drift
```

```bash
# Essential commands
terraform init              # Initialize backend/providers
terraform fmt -recursive    # Format all .tf files
terraform validate         # Check syntax
terraform plan -out=tfplan  # Dry-run
terraform apply tfplan      # Execute
terraform state list        # List managed resources
terraform state show <resource>  # Inspect resource
terraform import <resource> <id> # Import existing
terraform taint <resource>       # Force recreation (deprecated, use -replace)
terraform destroy -target=<resource>  # Destroy specific resource
```

## Pitfalls
- **State locking**: Always use remote backend with DynamoDB/locking
- **Drift**: Run `terraform plan` periodically to detect manual changes
- **Destroy order**: Dependencies can cause destroy failures — use `-target` for partial destroys
- **Variables in state**: Sensitive variables still visible in state file — encrypt at rest
- **Provider versions**: Pin versions to avoid breaking changes
- **Import**: Import alone doesn't generate config — use `terraform plan` to verify

## Verification
```bash
# Validate configuration
terraform validate
terraform fmt -check -recursive

# Check plan for unexpected changes
terraform plan -detailed-exitcode
echo $?  # 0=clean, 2=changes, 1=error

# Verify state integrity
terraform state pull | jq '.resources | length'
```