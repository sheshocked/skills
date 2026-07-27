---
name: cost-optimization-infra
description: 
category: devops
tags: [cost-optimization-infra]
---

## When to Use
Optimize cloud infrastructure costs: right-sizing instances, spot/reserved instances, storage tiering, unused resource cleanup, and cost monitoring. Covers AWS, GCP, and multi-cloud strategies.

## Core Concepts
- **Right-sizing**: Match instance type to actual workload
- **Reserved/Committed Use**: 1-3 year discounts for predictable workloads
- **Spot/Preemptible**: 60-90% discount for fault-tolerant workloads
- **Storage tiering**: Hot/warm/cold/archive based on access patterns
- **Auto-scaling**: Scale down during low traffic
- **Cost allocation**: Tags and budgets for per-team visibility

## Workflow
1. Audit current spend with cost explorer
2. Identify idle/unused resources
3. Right-size over-provisioned instances
4. Purchase reserved instances for baseline
5. Migrate fault-tolerant workloads to spot
6. Set up budget alerts

## Key Patterns
```bash
# AWS cost analysis
# Find idle EC2 instances (low CPU utilization)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-abc123 \
  --start-time $(date -u -d '30 days ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 86400 \
  --statistics Average | jq '.Datapoints[] | select(.Average < 10)'

# Find unattached EBS volumes
aws ec2 describe-volumes \
  --filters Name=status,Values=available \
  --query 'Volumes[*].[VolumeId,Size,CreateTime]' \
  --output table

# Find unused Elastic IPs
aws ec2 describe-addresses \
  --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' \
  --output table
```

```yaml
# Terraform — cost-optimized infrastructure
# Use spot instances for stateless workloads
resource "aws_launch_template" "api" {
  image_id      = "ami-abc123"
  instance_type = "c6g.large"  # ARM — 20% cheaper than x86

  instance_market_options {
    market_type = "spot"
    spot_options {
      spot_instance_type = "persistent"
      instance_interruption_behavior = "stop"
    }
  }

  tag_specifications {
    resource_type = "instance"
    tags = {
      Environment = "production"
      CostCenter  = "api-team"
    }
  }
}

# Auto-scaling group — scale to zero off-hours
resource "aws_autoscaling_group" "api" {
  min_size         = 1
  max_size         = 10
  desired_capacity = 2

  # Scale down at night
  scheduled_update_group_action {
    scheduled_action_name = "scale-down-night"
    min_size             = 1
    desired_capacity     = 1
    recurrence           = "0 22 * * *"
  }

  scheduled_update_group_action {
    scheduled_action_name = "scale-up-morning"
    min_size             = 2
    desired_capacity     = 3
    recurrence           = "0 6 * * 1-5"
  }
}

# S3 lifecycle for storage tiering
resource "aws_s3_bucket" "data" {
  bucket = "my-data-bucket"
}

resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    id = "tier-down"
    status = "Enabled"
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    transition {
      days          = 365
      storage_class = "DEEP_ARCHIVE"
    }
    expiration {
      days = 730  # Delete after 2 years
    }
  }
}
```

```bash
# Cost monitoring with budgets
# AWS Budgets — alert at 80% of budget
aws budgets create-budget \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --budget '{
    "BudgetName": "monthly-costs",
    "BudgetLimit": { "Amount": "5000", "Unit": "USD" },
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[
    {
      "Notification": { "NotificationType": "ACTUAL", "ComparisonOperator": "GREATER_THAN", "Threshold": 80 },
      "Subscribers": [{ "SubscriptionType": "EMAIL", "Address": "admin@example.com" }]
    }
  ]'
```

```bash
# GCP committed use discounts
# 1-year CUD: ~28% discount
# 3-year CUD: ~52% discount

# GCP preemption for batch workloads
gcloud compute instances create batch-worker \
  --provisioning-model=SPOT \
  --instance-termination-action=DELETE \
  --maintenance-policy=TERMINATE

# Find idle resources (GCP)
gcloud compute instances list \
  --filter="status=RUNNING" \
  --format="table(name, zone, machineType, networkInterfaces[0].networkIP)" | \
  while read name zone type ip; do
    cpu=$(gcloud compute instances get-serial-port-output "$name" --zone "$zone" 2>/dev/null | grep "cpu" | tail -1)
    echo "$name: $cpu"
  done
```

## Pitfalls
- **Over-provisioning**: Most instances run at <20% CPU — right-size aggressively
- **Reserved commitment**: Don't over-commit; use Convertible RIs for flexibility
- **Spot interruptions**: Design for interruption (checkpoint work, use multiple AZs)
- **Storage costs**: EBS/GPD volumes accumulate — delete unattached volumes
- **Data transfer costs**: Cross-region/AZ transfer is expensive; keep services in same AZ
- **Tagging**: Missing tags = no cost visibility; enforce tagging at provisioning

## Verification
```bash
# Review cost breakdown
aws ce get-cost-and-usage \
  --time-period Start=$(date -d '30 days ago' +%Y-%m-%d),End=$(date +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics "BlendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# Check reserved instance coverage
aws ce get-reserved-coverage \
  --time-period Start=$(date -d '30 days ago' +%Y-%m-%d),End=$(date +%Y-%m-%d)

# Verify spot usage
aws ec2 describe-instances \
  --filters Name=instance-lifecycle,Values=spot \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,SpotInstanceRequestId]' \
  --output table
```