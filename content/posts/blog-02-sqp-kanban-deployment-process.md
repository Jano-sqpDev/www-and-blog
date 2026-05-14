---
title: "SQP Kanban — The Deployment Process"
date: 2026-05-14
draft: false
tags: ["docker", "docker-compose", "github-actions", "devops", "ci-cd"]
description: "How I went from running a Node.js app locally to deploying it on my homelab server with Docker Compose and GitHub Actions."
---

This is the second post about containerising SQP Kanban. Here's how the actual process went.

## Step 1 — Dockerfile

Straightforward. Alpine-based Node image, copy `package.json` first (so `npm install` gets cached), then copy the app code. The main lesson was getting the layer order right for efficient rebuilds.

## Step 2 — Manual Docker Run

Before Docker Compose, I ran everything manually to understand what each piece does:

```bash
docker network create sqp-network
docker run -d --name sqp-mongo --network sqp-network -v sqp-mongo-data:/data/db mongo:8
docker build -t sqp-kanban .
docker run -d --name sqp-kanban --network sqp-network -p 4000:4000 -e MONGO_URI=mongodb://sqp-mongo:27017/sqp-kanban sqp-kanban
```

Four commands. Each one taught me something — networks, volumes, port mapping, environment variables. Worth doing manually at least once before reaching for Compose.

## Step 3 — Docker Compose

Replaced those four commands with a single `docker-compose.yml`. One gotcha — I'm on Fedora Cosmic (immutable), which uses Podman instead of Docker. The shorthand `build: .` syntax didn't work with `podman-compose`, so I had to use the expanded form with `context`, `dockerfile`, and an explicit `image` name. Volumes also needed `driver: local` declared explicitly. Both fixes are compatible with Docker too, so one file works everywhere.

## Step 4 — Self-hosted Runner

My homelab server (Bitfrost, Ubuntu) can't be reached from GitHub's cloud, so GitHub-hosted runners were out. I installed a self-hosted runner on Bitfrost instead — it connects outbound to GitHub and waits for jobs. No port forwarding needed.

Since the repo is public, I also set the fork PR policy to require approval for all external contributors. Without this, anyone could fork the repo and run code on my server.

## Step 5 — GitHub Actions Workflow

The workflow file is minimal — trigger on push to `main`, check out the code, run `docker compose up -d --build`. The first run failed because Bitfrost had Docker Compose v1 (uses a hyphen: `docker-compose`) instead of v2 (uses a space: `docker compose`). A quick `apt install docker-compose-plugin` fixed it.

## Step 6 — Testing the Pipeline

The dev workflow: work on `dev` branch, push, create a PR to `main`, merge. GitHub Actions picks it up and deploys to Bitfrost automatically. Tested by stopping containers, merging a PR, and confirming the app came back up with data intact.

## Takeaway

The whole setup took an afternoon. Most of the time was spent debugging — not writing config. The config files are simple; the errors along the way are where the real learning happens.
