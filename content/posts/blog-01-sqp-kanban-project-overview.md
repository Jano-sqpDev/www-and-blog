---
title: "SQP Kanban — Containerising a Node.js App with Docker"
date: 2026-05-14
draft: false
tags: ["docker", "docker-compose", "mongodb", "node", "devops", "github-actions"]
description: "Taking a Node.js kanban board app and containerising it with Docker, Docker Compose, and automated deployment via GitHub Actions."
---

I've been building a simple kanban board called SQP Kanban — a Node.js/Express app backed by MongoDB. The app itself was functional, but it was running bare on my machine with no containerisation, no CI/CD, and no deployment pipeline. Time to fix that.

## The App

SQP Kanban is a drag-and-drop project board with columns (To Do, In Progress, Completed, Omit), task cards with date tracking, and colour-coded projects. The frontend is vanilla HTML/JS, the backend is Express with Mongoose, and data lives in MongoDB.

## The Plan

The DevOps tasks I set out to do:

| Task | Purpose |
|------|---------|
| Write a Dockerfile | Package the app into a container |
| Create a docker-compose.yml | Run the app and MongoDB together |
| Set up a self-hosted GitHub Actions runner | Deploy automatically from my homelab server |
| Create a CI/CD pipeline | Merge to `main` triggers deployment to Bitfrost (my homelab Ubuntu server) |

## Architecture

The setup is two containers on one machine, managed by Docker Compose:

- **sqp-kanban** — the Node.js app, serves the frontend and API on port 4000
- **sqp-mongo** — MongoDB 8, data persisted in a named volume

A GitHub Actions workflow on a self-hosted runner handles deployment. When a PR is merged to `main`, the runner pulls the code and runs `docker compose up -d --build` on Bitfrost.

## What's Next

The next posts cover how the process went, what broke along the way, and the errors I had to debug — from Mongoose schema mismatches to browser APIs that only work on HTTPS.
