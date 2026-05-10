---
title: "Docker Basics — Building and Deploying to EC2"
date: 2026-05-08
draft: false
tags: ["docker", "aws", "ec2", "nginx", "podman"]
description: "Building a custom Docker image, pushing to Docker Hub, and running it on an AWS EC2 instance."
---

Docker packages an application and everything it needs into a container. Same image, identical behaviour on any machine. This post covers building a custom nginx image and deploying it to AWS EC2.

## Podman on Fedora

Docker doesn't run inside a Toolbox container — kernel module restrictions. Podman is Docker-compatible and works natively on Fedora:

```bash
echo 'alias docker=podman' >> ~/.bashrc
source ~/.bashrc
```

Same commands, no workarounds needed from this point.

## Building a Custom Image

```
docker-nginx/
├── Dockerfile
└── index.html
```

```dockerfile
FROM docker.io/library/nginx:latest
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

```bash
docker build -t my-nginx:v1.0 .
docker run -d -p 8080:80 --name my-nginx my-nginx:v1.0
```

Open `http://localhost:8080` — custom HTML served via nginx inside a container.

## Pushing to Docker Hub

```bash
docker login docker.io -u janosqpdev
docker tag my-nginx:v1.0 janosqpdev/my-nginx:v1.0
docker push janosqpdev/my-nginx:v1.0
```

Use an access token instead of a password — created in Docker Hub account settings.

## Deploying to EC2

Real Docker works perfectly on Ubuntu EC2 — no Podman workaround needed:

```bash
sudo apt update && sudo apt install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker

docker pull janosqpdev/my-nginx:v1.0
docker run -d -p 80:80 --name my-nginx janosqpdev/my-nginx:v1.0
```

Open `http://ec2-public-ip` — the same image built on Fedora, running on Ubuntu EC2.

## What I Learned

The container OS (Alpine, Ubuntu) comes from the image. The kernel always comes from the host machine. Same image runs on any Linux host — that's the whole point.

Next: Docker Compose to manage multiple containers together.
