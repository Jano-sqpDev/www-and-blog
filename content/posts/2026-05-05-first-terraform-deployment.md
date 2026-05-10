---
title: "First Terraform Deployment on AWS"
date: 2026-05-05
draft: false
tags: ["terraform", "aws", "ec2", "vpc"]
description: "Building my first AWS VPC and EC2 instance with Terraform — and the mistakes I made along the way."
---

With the environment set up, I wrote my first real Terraform code — a complete VPC with a public subnet, security group, and an EC2 instance running nginx.

## What I Built

- VPC (`10.0.0.0/16`)
- Public subnet with Internet Gateway
- Security group (SSH + HTTP)
- EC2 instance (Ubuntu 24.04)
- SSH key pair

## File Structure

```
exercise-02-variables-outputs/
├── main.tf
├── variables.tf
├── locals.tf
├── outputs.tf
├── data.tf
├── key.tf
└── terraform.tfvars
```

Separating resources into logical files keeps things clean and readable.

## Key Patterns

**Data source for AMI — no hardcoded IDs:**
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu-pro-server/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-pro-*"]
  }
}
```

**Locals for consistent naming:**
```hcl
locals {
  exercise    = "exer-02"
  name_prefix = "sqp-${local.exercise}"
}
```

## Mistakes I Made

**`.id` missing on resource references:**
```hcl
# Wrong — passes entire object
vpc_id = aws_vpc.vpc

# Correct — passes just the ID string
vpc_id = aws_vpc.vpc.id
```

**No public IP on EC2** — subnets don't auto-assign public IPs by default:
```hcl
map_public_ip_on_launch = true
```

**SSH port not open** — security groups are explicit, nothing is open by default. Port 22 needs its own ingress rule.

## Outputs

```hcl
output "ssh_command" {
  value = "ssh -i ~/.ssh/aws_key ubuntu@${aws_instance.server.public_ip}"
}
```

Small thing, big convenience — the exact SSH command printed after every `terraform apply`.

## Always Destroy When Done

```bash
terraform destroy
```

Even small EC2 instances add up. Good habit to build early.
