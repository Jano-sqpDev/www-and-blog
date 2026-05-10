---
title: "First Real Cloud Deployment"
date: 2026-05-10
draft: false
tags:
  - docker
  - aws
  - ec2
  - devops
  - postgres
  - caddy
categories:
  - DevOps
description: "Deploying a Dockerized full-stack application to AWS EC2 with PostgreSQL, Caddy, HTTPS and DuckDNS."
---

# First Real Cloud Deployment

Today I deployed my first real full-stack application to AWS EC2.

The project is a hospital staff allocation system built with:

- React + Vite frontend
- Express backend
- PostgreSQL database
- Docker containers
- Docker Compose orchestration

## What I Learned

The most important thing was understanding how the different layers connect together.

Final architecture:

```text
Browser
   ↓ HTTPS
DuckDNS
   ↓
Caddy reverse proxy
   ↓
Docker app container
   ↓
PostgreSQL container
```

## Deployment Workflow

The workflow looked like this:

```text
Local Fedora machine
   ↓
Build container image with Podman
   ↓
Push image to private Docker Hub repository
   ↓
EC2 pulls image
   ↓
Docker Compose starts containers
   ↓
Caddy provides HTTPS
```

## Biggest Learning Moments

I hit several real infrastructure problems during deployment:

- Docker socket permission issues
- Private Docker Hub authentication
- PostgreSQL password mismatches
- Database startup timing issues
- Reverse proxy configuration
- HTTPS setup

At first it felt chaotic, but debugging those issues taught me much more than simply following a tutorial.

## Important Realization

Infrastructure work is less about memorizing commands and more about understanding:

- which component is responsible
- where traffic flows
- how services communicate
- where state is stored
- how containers interact

## Final Result

The application now runs publicly with HTTPS:

```text
https://dsuopd.duckdns.org
```

with:
- automatic HTTPS certificates
- persistent PostgreSQL storage
- isolated containers
- cloud deployment
- reverse proxy architecture

This was probably the first time the concepts behind Docker, reverse proxies and deployment started feeling real instead of abstract.
