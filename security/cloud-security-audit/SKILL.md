---
name: cloud-security-audit
description: 
category: security
tags: [cloud-security-audit]
---

## When to Use
Use this skill when auditing cloud infrastructure security — AWS/GCP/Azure IAM policies, public S3 buckets, exposed credentials, security group misconfigurations, and compliance gaps.

## Core Concepts
- **IAM least privilege**: Policies should grant minimum required permissions
- **Public exposure**: S3 buckets, databases, and services accidentally made public
- **Credential leakage**: Access keys in code repos, environment variables, or logs
- **Security groups**: Overly permissive network ACLs (0.0.0.0/0 on management ports)
- **Logging**: CloudTrail/Cloud Audit Logs must be enabled and immutable
- **Encryption**: Data at rest and in transit must be encrypted

## Workflow
1. **Enumerate accounts**: Map all AWS/GCP accounts and subscriptions
2. **IAM audit**: Review policies for over-privilege and unused permissions
3. **Public exposure**: Scan for public S3 buckets, open databases, exposed APIs
4. **Credential scan**: Check code repos, logs, and config files for leaked keys
5. **Network review**: Audit security groups, VPC configurations, NACLs
6. **Logging check**: Verify CloudTrail, VPC Flow Logs, and audit logs are enabled
7. **Encryption check**: Verify EBS, RDS, S3 encryption is enabled
8. **Compliance report**: Map findings to CIS Benchmarks and compliance frameworks

## Key Patterns

### AWS IAM Audit
```bash
# Install and configure aws-cli
aws configure --profile audit-account

# List all IAM users and their access keys
aws iam list-users --query 'Users[*].[UserName,CreateDate,PasswordLastUsed]' --output table
aws iam list-access-keys --user-name TARGET_USER

# Find policies attached to a user
aws iam list-attached-user-policies --user-name TARGET_USER
aws iam list-user-policies --user-name TARGET_USER

# Check for wildcard permissions (dangerous)
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT:user/USER \
  --action-names s3:GetObject iam:CreateUser ec2:RunInstances

# Find all policies with wildcard actions
aws iam list-policies --scope Local --query 'Policies[*].[PolicyName,Arn]' --output text | \
while read name arn; do
  version=$(aws iam get-policy --policy-arn "$arn" --query 'Policy.DefaultVersionId' --output text)
  doc=$(aws iam get-policy-version --policy-arn "$arn" --version-id "$version" --query 'PolicyVersion.Document' --output json)
  if echo "$doc" | grep -q '"Action": "*"'; then
    echo "[DANGER] $name has wildcard actions"
  fi
done
```

### S3 Bucket Security Scan
```bash
# List all S3 buckets
aws s3api list-buckets --query 'Buckets[*].[Name,CreationDate]' --output table

# Check each bucket for public access
for bucket in $(aws s3api list-buckets --query 'Buckets[*].Name' --output text); do
  echo "=== $bucket ==="

  # Check public access block
  aws s3api get-public-access-block --bucket "$bucket" 2>/dev/null || echo "No public access block"

  # Check bucket policy
  aws s3api get-bucket-policy --bucket "$bucket" 2>/dev/null | jq -r '.Policy' | jq '.Statement[]'

  # Check ACL
  aws s3api get-bucket-acl --bucket "$bucket" | jq '.Grants[]'

  # Check encryption
  aws s3api get-bucket-encryption --bucket "$bucket" 2>/dev/null || echo "No default encryption"
done

# Automated scanner: ScoutSuite
pip install scoutsuite
scout aws --profile audit-account --report-dir /tmp/scout-report
```

### Credential Leak Detection
```bash
# Scan code repos for AWS keys
grep -rn "AKIA[0-9A-Z]\{16\}" . --include="*.py" --include="*.js" --include="*.yaml" --include="*.env"
grep -rn "aws_secret_access_key" . --include="*.py" --include="*.js" --include="*.yaml"

# Check git history for leaked keys
git log --all --oneline --diff-filter=D -- "*.env" "*.pem" "*.key"
git log --all -p | grep -i "AKIA[0-9A-Z]\{16\}"

# Use truffleHog for deep scan
pip install trufflehog
trufflehog git https://github.com/ORG/REPO --only-verified

# Check CloudTrail for credential usage from unexpected IPs
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin \
  --query 'Events[*].[EventTime,Username,CloudTrailEvent]' --output text

# Check for access keys in environment variables
env | grep -i "AWS\|SECRET\|TOKEN\|KEY"
```

### GCP Security Audit
```bash
# Install gcloud CLI
gcloud auth login --impersonate-service-account=audit@PROJECT.iam.gserviceaccount.com

# List all IAM bindings
gcloud projects get-iam-policy PROJECT_ID --format=json | jq '.bindings[] | select(.role | startswith("roles/"))'

# Check for public datasets in BigQuery
bq ls --format=json PROJECT_ID | jq -r '.[].id' | while read ds; do
  bq show --format=json "$ds" | jq '.access[]'
done

# Check GCS bucket permissions
gsutil ls -p PROJECT_ID | while read bucket; do
  echo "=== $bucket ==="
  gsutil iam get "$bucket"
done

# Scan for exposed APIs
gcloud services list --enabled --format=json | jq '.[].name'

# Check VPC firewall rules
gcloud compute firewall-rules list --format=json | \
  jq '.[] | select(.sourceRanges[] == "0.0.0.0/0") | {name, allowed, sourceRanges}'
```

### Automated Audit with Prowler (AWS)
```bash
# Install Prowler
pip install prowler

# Run full audit
prowler aws --profile audit-account --output-format html --output-directory /tmp/prowler-report

# Run specific checks
prowler aws --checks s3_bucket_public_access_enabled iam_root_access_key

# Check compliance
prowler aws --compliance cis_2.0_aws

# Export findings
prowler aws --output-format json --output-directory /tmp/prowler-findings
```

## Pitfalls
- **Cross-account access**: Audit roles that trust external accounts — these are high-risk
- **Lambda IAM roles**: Serverless functions often have overly broad roles — review per-function permissions
- **Default VPC**: Default VPCs often have public subnets with internet gateways — audit first
- **CloudTrail gaps**: Ensure trails cover all regions and management events
- **Cost implications**: Enabling full logging across all services can be expensive — prioritize critical accounts
- **Rotation**: Access keys should be rotated every 90 days — automate with IAM policies

## Verification
- All IAM policies follow least privilege (no wildcard actions in production)
- No public S3 buckets with sensitive data
- No leaked credentials in code repos or git history
- CloudTrail enabled in all regions with log file validation
- All EBS volumes and RDS instances have encryption enabled
- Security groups don't allow 0.0.0.0/0 on ports 22, 3389, 3306
- MFA enabled for all IAM users with console access
- Root account not used for daily operations