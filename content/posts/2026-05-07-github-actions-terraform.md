---
title: "Automating Terraform with GitHub Actions"
date: 2026-05-07
draft: false
tags: ["terraform", "github-actions", "cicd", "aws"]
description: "Setting up a CI/CD pipeline that automatically runs terraform plan on pull requests and terraform apply on merge."
---

Manually running `terraform apply` from a laptop works for learning. In the real world, infrastructure changes go through a pipeline. This is how I set one up.

## The Goal

- Every **Pull Request** → run `terraform plan` automatically
- Every **merge to main** → run `terraform apply` automatically
- AWS credentials stored securely as GitHub Secrets — never in code

## Workflow File

```yaml
name: Terraform CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: exercise-02-variables-outputs

    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/setup-terraform@v2
      - uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-west-2

      - run: terraform init
      - run: terraform fmt -check
      - run: terraform validate
      - run: terraform plan
        if: github.event_name == 'pull_request'
      - run: terraform apply -auto-approve
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
```

## Problems I Hit

**Workflow not showing in Actions tab** — GitHub only reads workflow files from the `main` branch. The file needs to be merged before it's recognised.

**`terraform fmt -check` failed** — inconsistent spacing in my files. Fix: always run `terraform fmt` locally before pushing.

**SSH public key not found** — `file("~/.ssh/aws_key.pub")` fails on GitHub's Ubuntu runner because the file doesn't exist there. Fix: store the public key as a GitHub Secret and pass it as an environment variable:

```yaml
env:
  TF_VAR_public_key: ${{ secrets.AWS_PUBLIC_KEY }}
```

Terraform automatically maps `TF_VAR_*` environment variables to input variables.

## GitOps in Practice

Once merged, every infrastructure change goes through the same path: code review → plan review → apply. The pipeline is the single source of truth.
