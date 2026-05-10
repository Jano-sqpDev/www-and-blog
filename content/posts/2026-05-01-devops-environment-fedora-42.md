---
title: "Setting Up a DevOps Environment on Fedora 42"
date: 2026-05-01
draft: false
tags: ["fedora", "toolbox", "devops", "setup"]
description: "How I set up my DevOps learning environment on Fedora 42 using Toolbox containers."
---

Coming from a VFX and IT background, I decided to transition into DevOps. This is the first post in a series documenting that journey.

## Why Fedora?

Fedora aligns well with enterprise DevOps — the same package manager (DNF), SELinux by default, and close ties to RHEL. The quirks you solve on Fedora build exactly the troubleshooting mindset DevOps engineers need.

## Toolbox for Isolation

Rather than installing everything directly on the host, I use [Toolbox](https://containertoolbx.org/) — a container-based environment built on Podman. My DevOps tools live in a container called `sandbox`, keeping the host clean.

```bash
toolbox create --container sandbox
toolbox enter sandbox
```

## Tools Installed Inside the Container

| Tool | Purpose |
|---|---|
| Terraform | Infrastructure as Code |
| AWS CLI | AWS service management |
| AWS SAM CLI | Serverless app deployment |
| Python 3 | Scripting |
| Git | Version control |
| VS Code | Editor (installed via RPM) |

```bash
# Terraform
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
sudo dnf install terraform -y

# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

aws configure
```

## Flatpak for GUI Apps

GUI apps like Chrome and Thunderbird run as Flatpaks on the host — sandboxed from the container environment.

```bash
flatpak install flathub com.google.Chrome
flatpak install flathub org.mozilla.Thunderbird
```

One gotcha — Flatpak apps are sandboxed and can't see host commands like `toolbox`. Worth knowing early.

## What's Next

With the environment ready, the next step was writing actual Terraform code and deploying infrastructure to AWS.
